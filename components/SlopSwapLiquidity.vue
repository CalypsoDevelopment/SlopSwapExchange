<template>
  <b-container fluid>
    <b-row class="trade-container">
      <b-col cols="12">
        <div>
          <b-nav>
            <b-nav-item active>
              <i class="fa-solid fa-gears" />
            </b-nav-item>
            <b-nav-item>
              <i class="fa-solid fa-clock-rotate-left" />
            </b-nav-item>
            <b-nav-item>
              <i class="fa-solid fa-chart-area" />
            </b-nav-item>
            <!--<b-nav-item>
            </b-nav-item>-->
          </b-nav>
        </div>
      </b-col>
      <b-col cols="5" class="text-center">
        <SlopSwapMakerTokenSelect />
      </b-col>
      <b-col cols="2" class="text-center">
        <div class="align-middle trade-symbol-container">
          <i class="fa-solid fa-repeat fa-4x animate__animated animate__rotatIn" style="color: #FFFFFF" />
          <b-form-select v-model="SlippageSelected" class="slippage-selector mt-2" :options="SlippageOptions" />
        </div>
      </b-col>
      <b-col cols="5" class="text-center">
        <SlopSwapTakerTokenSelect />
      </b-col>
    </b-row>
  </b-container>
</template>
<script>
import axios from 'axios'
import SlopSwapMakerTokenSelect from '~/components/SlopSwapMakerTokenSelect.vue'
import SlopSwapTakerTokenSelect from '~/components/SlopSwapTakerTokenSelect.vue'
const ethers = require('ethers')
const qs = require('qs')
const BEP20 = require('~/static/artifacts/IERC20_metadata.json')
const PAIR = require('~/static/artifacts/SlopSwapPair.json')

// SlopSwap Factory 0x0533B75362E3Be13E78f245e50674c9a6dd9c17A
// SlopSwap Router 0x42A77DEdD13520141aaF1720EF88086B5Cae95f5
// PancakeSwap Factory 0xcA143Ce32Fe78f1f7019d7d551a6402fC5350c73
// Pancake Router 0x10ED43C718714eb63d5aA57B78B54704E256024E

export default {
  name: 'SlopSwapLiquidity',
  components: { SlopSwapMakerTokenSelect, SlopSwapTakerTokenSelect },
  data () {
    return {
      wbnbcake: null
    }
  },
  mounted () {
    // URL connection
    /* const kline = new WebSocket('wss://dex.binance.org/api/ws/WBNB_CAKE-1DE@kline_1h')
    alert(kline)
    this.wbnbcake = kline

    // Or Subscribe method
    const conn = new WebSocket('wss://dex.binance.org/api/ws')
    conn.onopen = function (evt) {
      conn.send(JSON.stringify({ method: 'subscribe', topic: 'kline_1h', symbols: ['BNB_BTCB-1DE'] }))
    } */
  },
  methods: {
    async CheckForLiquidityTokens () {
      // Establish the connection to the User wallet & query Token A (Primary Liquidity Token) balance within the wallet
      // A Web3Provider wraps a standard Web3 provider, which ism
      // what MetaMask injects as window.ethereum into each page
      const provider = new ethers.providers.Web3Provider(window.ethereum)

      // MetaMask requires requesting permission to connect users accounts
      await provider.send('eth_requestAccounts', [])

      // The MetaMask plugin also allows signing transactions to
      // send ether and pay to change state within the blockchain.
      // For this, you need the account signer...
      const signer = provider.getSigner()
      // alert('Signer: ' + signer)

      // alert('Before Account Request')
      // const account = await window.ethereum.request({ method: 'eth_requestAccounts' })
      // alert('After Account Request: ' + account)

      const address1 = this.TokenA.TokenContract
      const address2 = this.TokenB.TokenContract

      const factory = new ethers.Contract(String(this.MainnetFactory), this.MainnetFactoryABI, signer)
      // const router = new ethers.Contract(String(this.MainnetRouter), this.MainnetRouterABI, this.signer)
      const pairAddress = await factory.getPair(address1, address2)

      alert('Pair Address: ' + pairAddress)
      const LPTokenContract = new ethers.Contract(String(this.MainnetPair), this.MainnetPairABI, signer)
      // alert('After Creation of LP Token Contract Instance')

      const BEP20TokenContract = new ethers.Contract(pairAddress, BEP20, provider)

      const LPTokenBalance = await BEP20TokenContract.balanceOf(String('0x0684e0e2d497442d848666122ababa2a28d0eca7'))
      this.LiquidityBalance = ethers.utils.formatEther(String(LPTokenBalance))
      const LPTokenName = await LPTokenContract.name()
      const LPTokenTotalSupply = await LPTokenContract.totalSupply()
      alert(LPTokenName + ' Total Supply: ' + ethers.utils.formatEther(String(LPTokenTotalSupply)))
      alert('LP Token Name: ' + LPTokenName)
      const LPTokenFormatter = ethers.utils.formatEther(String(LPTokenBalance))
      alert('SlopSwap LP Token Balance: ' + LPTokenFormatter + '\n' + 'LP Token Address: ' + pairAddress)
    },
    async quoteRemoveLiquidity (
    ) {
      // Function used to get a quote of the liquidity removal
      //    `address1` - An Ethereum address of the coin to recieve (either a token or AUT)
      //    `address2` - An Ethereum address of the coin to recieve (either a token or AUT)
      //    `liquidity` - The amount of liquidity tokens the user will burn to get their tokens back
      //    `factory` - The current factory
      //    `signer` - The current signer

      // Establish the connection to the User wallet & query Token A (Primary Liquidity Token) balance within the wallet
      // A Web3Provider wraps a standard Web3 provider, which ism
      // what MetaMask injects as window.ethereum into each page
      const provider = new ethers.providers.Web3Provider(window.ethereum)

      // MetaMask requires requesting permission to connect users accounts
      await provider.send('eth_requestAccounts', [])

      // The MetaMask plugin also allows signing transactions to
      // send ether and pay to change state within the blockchain.
      // For this, you need the account signer...
      const signer = provider.getSigner()
      // alert('Signer: ' + signer)

      const account = await window.ethereum.request({ method: 'eth_requestAccounts' })
      this.UserAccount = account.address

      const liquidity = this.LiquidityBalance

      const factory = new ethers.Contract(String(this.MainnetFactory), this.MainnetFactoryABI, signer)
      // const router = new ethers.Contract(String(this.MainnetRouter), this.MainnetRouterABI, this.signer)

      const address1 = this.TokenA.TokenContract
      const address2 = this.TokenB.TokenContract

      const pairAddress = await factory.getPair(String(address1), String(address2))
      alert('Pair Address: ' + pairAddress)
      const pair = new ethers.Contract(this.MainnetPair, PAIR, signer)
      // alert('After Creating Pair Contract Instance')

      // alert('Before reserves Raw request')
      const reservesRaw = await pair.getReserves() // Returns the reserves already formated as ethers
      const reserveA = reservesRaw[0]
      const reserveB = reservesRaw[1]
      alert('Reserves Raw Token A: ' + reservesRaw[0] + ' ' + 'Reserves Raw Token B: ' + reservesRaw[1])

      const feeOn =
        (await factory.feeTo()) !== 0x0000000000000000000000000000000000000000

      const _kLast = await pair.kLast()
      // alert('kLast: ' + _kLast)
      const kLast = Number(ethers.utils.formatEther(_kLast))

      const _totalSupply = await pair.totalSupply()
      alert('Total Supply: ' + _totalSupply)
      let totalSupply = Number(ethers.utils.formatEther(_totalSupply))

      if (feeOn && kLast > 0) {
        const feeLiquidity =
          (totalSupply * (Math.sqrt(reserveA * reserveB) - Math.sqrt(kLast))) /
          (5 * Math.sqrt(reserveA * reserveB) + Math.sqrt(kLast))
        totalSupply = totalSupply + feeLiquidity
      }

      const Aout = (reserveA * liquidity) / totalSupply
      const Bout = (reserveB * liquidity) / totalSupply

      alert('Aout: ' + Aout + ' ' + 'Bout: ' + Bout + ' ' + 'Liquidity: ' + liquidity)
      // return [liquidity, Aout, Bout]
      this.liquidity = liquidity
      this.Aout = Aout
      this.Bout = Bout
    },
    async RemoveLiquidity (
      LiquidityTokens
    ) {
      // Establish the connection to the User wallet & query Token A (Primary Liquidity Token) balance within the wallet
      // A Web3Provider wraps a standard Web3 provider, which ism
      // what MetaMask injects as window.ethereum into each page
      const provider = new ethers.providers.Web3Provider(window.ethereum)

      // MetaMask requires requesting permission to connect users accounts
      await provider.send('eth_requestAccounts', [])

      // The MetaMask plugin also allows signing transactions to
      // send ether and pay to change state within the blockchain.
      // For this, you need the account signer...
      const signer = provider.getSigner()
      // alert('Signer: ' + signer)

      const account = await window.ethereum.request({ method: 'eth_requestAccounts' })
      this.UserAccount = account.address

      const address1 = this.TokenA.TokenContract
      const address2 = this.TokenB.TokenContract

      const pair = new ethers.Contract(String(this.MainnetPair), PAIR, signer)
      const factory = new ethers.Contract(String(this.MainnetFactory), this.MainnetFactoryABI, signer)
      const router = new ethers.Contract(String(this.MainnetRouter), this.MainnetRouterABI, signer)

      const pairAddress = await factory.getPair(String(address1), String(address2))
      alert('Pair Address: ' + pairAddress)
      // const pairsLength = await factory.allPairsLength() // Returns the reserves already formated as ethers
      // alert('Pairs Length: ' + pairsLength)

      const token1 = new ethers.Contract(String(address1), BEP20, signer)
      alert('Token1: ' + token1.address)
      const token2 = new ethers.Contract(String(address2), BEP20, signer)
      alert('Token2: ' + token2.address)

      const token1Decimals = this.TokenA.TokenDecimal
      const token2Decimals = this.TokenB.TokenDecimal
      alert('After Token Decimals')

      const Getliquidity = (LiquidityTokens) => {
        if (LiquidityTokens < 0.001) {
          return ethers.BigNumber.from(LiquidityTokens * 10 ** 18)
        }
        return ethers.utils.parseUnits(String(LiquidityTokens), 18)
      }

      const liquidity = Getliquidity(LiquidityTokens)
      alert('liquidity: ' + liquidity)

      const amount1 = this.Aout
      const amount2 = this.Bout

      const SubtractorA = amount1 * this.SlippageSelected
      const SubtractorANum = amount1 - SubtractorA
      const MinA = ethers.utils.parseUnits(String(SubtractorANum), this.TokenA.TokenADecimals)
      // alert(MinA)

      const SubtractorB = amount2 * this.SlippageSelected
      const SubtractorBNum = amount2 - SubtractorB
      const MinB = ethers.utils.parseUnits(String(SubtractorBNum), this.TokenA.TokenADecimals)
      // alert(MinB)

      const amount1Min = ethers.utils.parseUnits(String(MinA), token1Decimals)
      const amount2Min = ethers.utils.parseUnits(String(MinB), token2Decimals)
      // alert('After AmountMin')
      const time = Math.floor(Date.now() / 1000) + 200000
      const deadline = ethers.BigNumber.from(time)

      const wethAddress = await router.WETH()
      // const pairAddress = await factory.getPair(address1, address2);
      // const pair = new Contract(pairAddress, PAIR.abi, signer);
      alert('Before pair.approve')
      await pair.approve(router.address, liquidity)
      alert('After pair.approve')

      alert([
        String(address1),
        String(address2),
        Number(liquidity),
        Number(amount1Min),
        Number(amount2Min),
        String(account),
        deadline
      ])

      if (address1 === wethAddress) {
        // Eth + Token
        await router.removeLiquidityETH(
          String(address2),
          String(liquidity),
          String(amount2Min),
          String(amount1Min),
          String(account),
          deadline
        )
      } else if (address2 === wethAddress) {
        // Token + Eth
        await router.removeLiquidityETH(
          String(address1),
          String(liquidity),
          String(amount1Min),
          String(amount2Min),
          String(account),
          deadline
        )
      } else {
        // Token + Token
        await router.removeLiquidity(
          String(address1),
          String(address2),
          String(liquidity),
          String(amount1Min),
          String(amount2Min),
          String(account),
          deadline
        )
      }
    },
    async RetrieveUserBalances () {
      // Define Token A & B
      // Establish the connection to the User wallet & query Token A (Primary Liquidity Token) balance within the wallet
      // A Web3Provider wraps a standard Web3 provider, which ism
      // what MetaMask injects as window.ethereum into each page
      const provider = new ethers.providers.Web3Provider(window.ethereum)

      // MetaMask requires requesting permission to connect users accounts
      await provider.send('eth_requestAccounts', [])

      // The MetaMask plugin also allows signing transactions to
      // send ether and pay to change state within the blockchain.
      // For this, you need the account signer...
      // const signer = provider.getSigner()
      // alert('Signer: ' + signer)

      const account = await window.ethereum.request({ method: 'eth_requestAccounts' })
      // this.UserAccount = account.address
      // alert('Wallet User: ' + account)
      // alert('Before Token Symbol')

      if (this.TokenA.TokenSymbol === 'WBNB') {
        // Get Token Balance through Metamask
        const TokenABalance = await provider.getBalance(String(account))
        // alert(TokenABalance)
        const ReturnTokenABalance = ethers.utils.formatEther(String(TokenABalance))
        // alert('User TokenA Balance: ' + ConvertWeiToEther + ' WBNB')

        this.TokenAUserBalance = ReturnTokenABalance.substring(0, 6) + ' ' + this.TokenA.TokenSymbol
      } else {
        const BEP20TokenA = new ethers.Contract(
          this.TokenA.TokenContract, [
            'function name() view returns (string)',
            'function symbol() view returns (string)',
            'function balanceOf(address) view returns (uint)'
          ],
          provider
        )
        const TokenAbalance = await BEP20TokenA.balanceOf(String(account))
        // alert(TokenAbalance)
        const ReturnTokenAbalance = ethers.utils.formatUnits(String(TokenAbalance), this.TokenA.TokenDecimal)
        this.TokenAUserBalance = ReturnTokenAbalance.substring(0, 8) + ' ' + this.TokenA.TokenSymbol
      }

      const BEP20TokenB = new ethers.Contract(
        this.TokenB.TokenContract, [
          'function name() view returns (string)',
          'function symbol() view returns (string)',
          'function balanceOf(address) view returns (uint)'
        ],
        provider
      )
      const TokenBbalance = await BEP20TokenB.balanceOf(String(account))
      // alert(TokenBbalance)
      const ReturnTokenBbalance = ethers.utils.formatUnits(String(TokenBbalance), this.TokenB.TokenDecimal)
      this.TokenBUserBalance = ReturnTokenBbalance.substring(0, 8) + ' ' + this.TokenB.TokenSymbol
    },
    // Function used to add Liquidity to any pair of tokens or token-AUT
    // To work correctly, there needs to be 9 arguments:
    //    `address1` - An Ethereum address of the coin to add from (either a token or AUT)
    //    `address2` - An Ethereum address of the coin to add to (either a token or AUT)
    //    `amount1` - A float or similar number representing the value of address1's coin to add
    //    `amount2` - A float or similar number representing the value of address2's coin to add
    //    `amount1Min` - A float or similar number representing the minimum of address1's coin to add
    //    `amount2Min` - A float or similar number representing the minimum of address2's coin to add
    //    `routerContract` - The router contract to carry out this trade
    //    `accountAddress` - An Ethereum address of the current user's account
    //    `provider` - The current provider
    //    `signer` - The current signer
    async addLiquidity () {
      // A Web3Provider wraps a standard Web3 provider, which is
      // what MetaMask injects as window.ethereum into each page
      const provider = new ethers.providers.Web3Provider(window.ethereum)

      // MetaMask requires requesting permission to connect users accounts
      await provider.send('eth_requestAccounts', [])

      // The MetaMask plugin also allows signing transactions to
      // send ether and pay to change state within the blockchain.
      // For this, you need the account signer...
      const signer = provider.getSigner()
      this.signer = signer

      const account = await window.ethereum.request({ method: 'eth_requestAccounts' })
      this.account = account
      alert(this.account)
      // const factory = new ethers.Contract(String(this.MainnetFactory), this.MainnetFactoryABI, signer)

      const address1 = this.TokenA.TokenContract
      const address2 = this.TokenB.TokenContract
      alert('After Token Address Assign Line 256')
      const RouterContract = new ethers.Contract(String(this.MainnetRouter), this.MainnetRouterABI, this.signer)

      const TokenAContract = new ethers.Contract(String(address1), BEP20, this.signer)
      const TokenBContract = new ethers.Contract(String(address2), BEP20, this.signer)
      alert('After Token Contact Instance Creation Line 261')

      const TokenADecimals = this.TokenA.TokenDecimal
      const TokenBDecimals = this.TokenB.TokenDecimal
      alert('TokenADecimals & TokenBDecimals ' + TokenADecimals + ' ' + TokenBDecimals + ' Line 265')

      const amount1 = this.TokenAPairAmount
      const amount2 = this.TokenBPairAmount

      alert('Amount1 ' + amount1)
      alert('Amount2 ' + amount2)

      const amountIn1 = ethers.utils.parseUnits(amount1, TokenADecimals)
      const amountIn2 = ethers.utils.parseUnits(amount2, TokenBDecimals)

      /* const amount1Min = ethers.utils.parseUnits(String(amount1), TokenADecimals)
      const amount2Min = ethers.utils.parseUnits(String(amount2), TokenBDecimals) */

      const time = Math.floor(Date.now() / 1000) + 200000
      const deadline = ethers.BigNumber.from(time)

      await TokenAContract.approve(String(this.MainnetRouter), String(amountIn1))
      await TokenBContract.approve(String(this.MainnetRouter), String(amountIn2))

      const wethAddress = await RouterContract.WETH()
      alert('WETH Address from Router Contract: ' + wethAddress)

      const SubtractorA = amount1 * this.SlippageSelected
      const SubtractorANum = amount1 - SubtractorA
      const FinalMinA = ethers.utils.parseUnits(String(SubtractorANum), this.TokenA.TokenADecimals)
      alert(FinalMinA)

      const SubtractorB = amount2 * this.SlippageSelected
      const SubtractorBNum = amount2 - SubtractorB
      const FinalMinB = ethers.utils.parseUnits(String(SubtractorBNum), this.TokenA.TokenADecimals)
      alert(FinalMinB)

      // const factory = new ethers.Contract(String(this.MainnetFactory), this.MainnetFactoryABI, signer)
      // const pairAddress = await factory.getPair(address1, address2)

      alert([
        'Address 1: ' + address1,
        'Address 2: ' + address2,
        'Amount In 1: ' + amountIn1,
        'Amount In 2: ' + amountIn2,
        'Amount 1 Min: ' + FinalMinA,
        'Amount 2 Min: ' + FinalMinB,
        'This Account: ' + this.account,
        'Deadline' + deadline
      ])

      if (address1 === wethAddress) {
        // Eth + Token
        await RouterContract.addLiquidityETH(
          String(address2),
          String(amountIn2),
          String(FinalMinB),
          String(FinalMinA),
          String(this.account),
          String(deadline),
          { value: String(amountIn1) }
        )
      } else if (address2 === wethAddress) {
        // Token + Eth
        await RouterContract.addLiquidityETH(
          String(address1),
          String(amountIn1),
          String(FinalMinA),
          String(FinalMinB),
          String(this.account),
          String(deadline),
          { value: String(amountIn2) }
        )
      } else {
        // Token + Token
        await RouterContract.addLiquidity(
          String(address1),
          String(address2),
          String(amountIn1),
          String(amountIn2),
          String(FinalMinA),
          String(FinalMinB),
          String(this.account),
          String(deadline)
        )
      }
    },
    ChangePairTokenA (TokenName, TokenSymbol, TokenContract) {
      this.TokenA = { TokenName, TokenSymbol, TokenContract }
      this.CheckTradingPair()
      this.$bvModal.hide('TokenA')
    },
    ChangePairTokenB (TokenName, TokenSymbol, TokenContract) {
      this.TokenB = { TokenName, TokenSymbol, TokenContract }
      this.CheckTradingPair()
      this.$bvModal.hide('TokenB')
    },
    async CheckTradingPair () {
      // A Web3Provider wraps a standard Web3 provider, which is
      // what MetaMask injects as window.ethereum into each page
      const provider = new ethers.providers.Web3Provider(window.ethereum)

      // MetaMask requires requesting permission to connect users accounts
      await provider.send('eth_requestAccounts', [])

      // The MetaMask plugin also allows signing transactions to
      // send ether and pay to change state within the blockchain.
      // For this, you need the account signer...
      const signer = provider.getSigner()

      const TokenA = this.TokenA.TokenContract
      // alert('TokanA Ref. ' + TokenA)
      const TokenB = this.TokenB.TokenContract
      // alert('TokenB Ref. ' + TokenB)

      const factoryInstance = new ethers.Contract(this.MainnetFactory, this.MainnetFactoryABI, signer)
      // alert('factory Ref.' + factoryInstance)
      this.factory = factoryInstance

      const routerInstance = new ethers.Contract(this.MainnetRouter, this.MainnetRouterABI, signer)
      // alert('router Ref.' + routerInstance)
      this.router = routerInstance

      const WBNBInstance = new ethers.Contract(this.TokenA.TokenContract, this.WBNBABI, signer)
      // alert('WBNBInstance ' + WBNBInstance)
      this.WBNBConInst = WBNBInstance

      const BEP20ContractInstance = new ethers.Contract(
        this.TokenB, [
          'function name() view returns (string)',
          'function symbol() view returns (string)',
          'function balanceOf(address) view returns (uint)'
        ],
        signer
      )
      this.BEP20ConInst = BEP20ContractInstance

      // alert('BEP20ContractInstance Contract Instance ' + BEP20ContractInstance)

      const checkPairs = await this.factory.getPair(TokenA, TokenB)
      // alert('Check If the Trading Pair Exists ' + checkPairs)
      this.PairCheckResp = checkPairs
      if (this.PairCheckResp === '0x0000000000000000000000000000000000000000') {
        // alert('That Trading Pair does not exist. You can be the first to start a trading pool with the pair')
      } else {
        // alert('This Trading Pair already exists, and the address is ' + this.PairCheckResp)
      }
      // alert(this.PairCheckResp)
    },
    async PairQuoteCheck () {
      const TokenAFormatted = ethers.utils.parseUnits(this.TokenAPairAmount, this.TokenA.TokenDecimal)
      // alert(priceFormatter)
      /* const QuoteReview = `
      Adding Liquidity to Token Pair - ${this.TokenA.TokenSymbol}/${this.TokenB.TokenSymbol}
      ${TokenAFormatted}
      =================`
       alert(QuoteReview) */

      const params = {
        // Not all token symbols are supported. The address of the token can be used instead.
        sellToken: this.TokenA.TokenContract,
        buyToken: this.TokenB.TokenContract,
        // Note that the DAI token uses 18 decimal places, so `sellAmount` is `100 * 10^18`.
        sellAmount: String(TokenAFormatted)
      }

      const response = await axios.get(
    `https://bsc.api.0x.org/swap/v1/quote?${qs.stringify(params)}`
      )
      this.PairQuote = response
      // alert(this.PairQuote)
      this.TokenBPairAmount = ethers.utils.formatUnits(String(this.PairQuote.data.buyAmount), this.TokenB.TokenDecimal)

      // Units.convert(String(this.quote.data.buyAmount), 'wei', 'ether')
    },
    async CreateTradingPair () {
      /* alert(`
          __________________________________________
          We will now create a Trading Pair for:
          ${this.TokenA.TokenContract} | ${this.TokenB.TokenContract}
          __________________________________________`) */
      const createTradingPair = await this.factory.createPair(this.TokenA.TokenContract, this.TokenB.TokenContract)
      const receipt = await createTradingPair.wait()
      this.CreateReceipt = receipt

      alert(`Transaction receipt : https://www.bscscan.com/tx/${receipt.transactionHash}`)
      this.CreatePairReceipt = receipt.logs
      this.PairCheckResp = receipt.logs + '<br>' + `Transaction receipt : <a href="https://www.bscscan.com/tx/${receipt.transactionHash}" target="_blank" title="${this.TokenA.TokenName} / ${this.TokenB.TokenName} Trading Pair Creation Transaction Receipt from SlopSwap Exchange">View Transaction on BSCscan</a>`
    },
    async CheckPair () {
      // A Web3Provider wraps a standard Web3 provider, which is
      // what MetaMask injects as window.ethereum into each page
      const provider = new ethers.providers.Web3Provider(window.ethereum)

      // MetaMask requires requesting permission to connect users accounts
      await provider.send('eth_requestAccounts', [])

      // The MetaMask plugin also allows signing transactions to
      // send ether and pay to change state within the blockchain.
      // For this, you need the account signer...
      const signer = provider.getSigner()

      const TokenA = this.TokenA.TokenContract
      // alert('TokanA Ref. ' + TokenA)
      const TokenB = this.TokenB.TokenContract
      // alert('TokenB Ref. ' + TokenB)

      const factoryInstance = new ethers.Contract(this.MainnetFactory, this.MainnetFactoryABI, signer)
      // alert('factory Ref.' + factoryInstance)
      this.factory = factoryInstance

      const routerInstance = new ethers.Contract(this.MainnetRouter, this.MainnetRouterABI, signer)
      // alert('router Ref.' + routerInstance)
      this.router = routerInstance

      const WBNBInstance = new ethers.Contract(this.TokenA.TokenContract, this.WBNBABI, signer)
      // alert('WBNBInstance ' + WBNBInstance)
      this.WBNBConInst = WBNBInstance

      const BEP20ContractInstance = new ethers.Contract(
        this.TokenB, [
          'function name() view returns (string)',
          'function symbol() view returns (string)',
          'function balanceOf(address) view returns (uint)'
        ],
        signer
      )
      this.BEP20ConInst = BEP20ContractInstance

      // alert('BEP20ContractInstance Contract Instance ' + BEP20ContractInstance)

      const checkPairs = await this.factory.getPair(TokenA, TokenB)
      // alert('Check If the Trading Pair Exists ' + checkPairs)
      this.PairCheckResp = checkPairs
      if (this.PairCheckResp === '0x0000000000000000000000000000000000000000') {
        // alert('That Trading Pair does not exist. You can be the first to start a trading pool with the pair')
        return false
      } else {
        // alert('This Trading Pair already exists, and the address is ' + this.PairCheckResp)
        return this.PairCheckResp
      }
      // alert(this.PairCheckResp)
    } // END OF CHECKPAIR

  } // END OF METHODS
}
</script>
<style scoped>

</style>
