<template>
  <b-container fluid>
    <b-row>
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
          </b-nav>
        </div>
      </b-col>
      <b-col cols="5" class="text-center">
        <SlopSwapMakerTokenSelect @changeMakerToken="ChangeSellToken($event)" />
        <div>
          <b-form-input v-model="sellAmount" class="maker-token-amount" placeholder="0.0" @change="MakerPriceCheck()" />
          <div class="mt-2 dollar-value">
            <i class="fa-solid fa-square-dollar" style="color: #6c757d;" />
            {{ MakerDollarAmount }}
          </div>
        </div>
      </b-col>
      <b-col cols="2" class="text-center">
        <div class="align-middle trade-symbol-container">
          <i class="fa-solid fa-repeat fa-4x animate__animated animate__rotatIn" style="color: #FFFFFF" />
        </div>
      </b-col>
      <b-col cols="5" class="text-center">
        <SlopSwapTakerTokenSelect @changeTakerToken="ChangeBuyToken($event)" />
        <div>
          <b-form-input v-model="BuyTokenAmount" class="taker-token-amount" placeholder="0.0" />
          <div class="mt-2 dollar-value">
            <i class="fa-solid fa-square-dollar" style="color: #6c757d;" />
            {{ TakerDollarAmount }}
          </div>
        </div>
      </b-col>
      <b-col cols="3" />
      <b-col cols="6">
        <b-button
          pill
          block
          class="trade-btn"
          variant="primary"
        >
          Trade Tokens
        </b-button>
      </b-col>
      <b-col cols="3" />
    </b-row>
    <b-row>
      <b-col />
    </b-row>
  </b-container>
</template>
<script>
import SlopSwapMakerTokenSelect from '~/components/SlopSwapMakerTokenSelect.vue'
import SlopSwapTakerTokenSelect from '~/components/SlopSwapTakerTokenSelect.vue'
const axios = require('axios')
const ethers = require('ethers')
const qs = require('qs')

export default {
  name: 'SlopSwapTrader',
  components: { SlopSwapMakerTokenSelect, SlopSwapTakerTokenSelect },
  data () {
    return {
      sellToken: { ChainID: 56, TokenName: 'Wrapped BNB', TokenSymbol: 'WBNB', TokenContract: '0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c', TokenDecimal: 18, TokenType: 'ERC20', BrandPrimary: '#f0b90b' },
      sellAmount: null,
      buyToken: { ChainID: 56, TokenName: 'PancakeSwap Token (Cake)', TokenSymbol: 'Cake', TokenContract: '0x0E09FaBB73Bd3Ade0a17ECC321fD13a19e81cE82', TokenDecimal: 18, TokenType: 'ERC20', BrandPrimary: '#d1884f' },
      buyAmount: null,
      quote: null,
      SellTokenAmount: null,
      BuyTokenAmount: null,
      MakerDollarAmount: 0.00,
      TakerDollarAmount: 0.00
    }
  },
  methods: {
    async MakerPriceCheck () {
      const priceFormatter = ethers.utils.parseUnits(String(this.sellAmount), this.sellToken.TokenDecimal)
      alert(priceFormatter)
      const QuoteReview = `
      Buying ${this.buyToken.TokenSymbol} Contract: ${this.buyToken.TokenContract}  using ${this.sellToken.TokenSymbol} in the amount of
      ${priceFormatter}
      =================`
      alert(QuoteReview)

      const params = {
        // Not all token symbols are supported. The address of the token can be used instead.
        sellToken: this.sellToken.TokenContract,
        buyToken: this.buyToken.TokenContract,
        // Note that the DAI token uses 18 decimal places, so `sellAmount` is `100 * 10^18`.
        sellAmount: String(priceFormatter)
      }

      const response = await axios.get(
    `https://bsc.api.0x.org/swap/v1/quote?${qs.stringify(params)}`
      )
      this.quote = response
      // alert(this.quote)
      // const BuyAmountWei = this.quote.data.buyAmount
      this.BuyTokenAmount = ethers.utils.formatUnits(String(this.quote.data.buyAmount), this.buyToken.TokenDecimal)

      // Units.convert(String(this.quote.data.buyAmount), 'wei', 'ether')
    },
    ChangeSellToken (MakerToken) {
      this.sellToken = MakerToken
      alert('The Sell Token has been changed to \nChainID ' + this.sellToken.ChainID + '\nSell Token Name:  ' + this.sellToken.TokenName + ' \nSell Token Symbol: ' + this.sellToken.TokenSymbol + '\nSell Token Contract: ' + this.sellToken.TokenContract + '\nSell Token Decimal: ' + this.sellToken.TokenDecimal + '\nSell Token Type: ' + this.buyToken.TokenType)
      this.sellAmount = null
      this.buyAmount = null
      this.BuyTokenAmount = null
    },
    ChangeBuyToken (TakerToken) {
      this.buyToken = TakerToken
      alert('The Buy Token has been changed to \nChainID: ' + this.buyToken.ChainID + '\nBuy Token Name: ' + this.buyToken.TokenName + '\nBuy Token Symbol: ' + this.buyToken.TokenSymbol + '\nBuy Token Contract: ' + this.buyToken.TokenContract + '\nBuy Token Decimal: ' + this.buyToken.TokenDecimal + '\nBuy Token Type: ' + this.buyToken.TokenType)
      this.sellAmount = null
      this.buyAmount = null
      this.BuyTokenAmount = null
    }
  }
}
</script>
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Arimo:ital,wght@0,400;0,500;0,600;0,700;1,400;1,500;1,600;1,700&display=swap');
.trade-btn {
  padding: .25rem;
  margin-top: 2rem;
  font-size: 1.6rem;
  font-family: 'Arimo', sans-serif;
  font-variant: all-small-caps;
}
.maker-token-amount {
  border-radius: 4rem;
  margin-top: 1rem;
}
.taker-token-amount {
  border-radius: 4rem;
  margin-top: 1rem;
}
.dollar-value {
  font-size: 1.8rem;
}
.trade-symbol-container {
  margin-top: 5rem;
  margin-bottom:2rem;
}
.animate__animated.animate__rotatIn {
  --animate-duration: 2s;
}
</style>
