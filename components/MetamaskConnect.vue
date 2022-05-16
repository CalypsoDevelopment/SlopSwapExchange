<template>
  <div />
</template>
<script>
const ethers = require('ethers')
const slop = require('slopswapxlibs')

export default {
  name: 'MetamaskConnect',
  components: {},
  data () {
    return {
      MetamaskConnection: Object
    }
  },
  beforeMount () {
    this.connectMeta()
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
    async connectMeta () {
      const provider = slop.getProvider()
      // MetaMask requires requesting permission to connect users accounts
      const accounts = await provider.send('eth_requestAccounts', [])
      const account = accounts[0]
      // The MetaMask plugin also allows signing transactions to
      // send ether and pay to change state within the blockchain.
      // For this, you need the account signer...
      const signer = slop.getSigner(provider)

      this.MetamaskConnection = {
        provider,
        signer,
        account
      }
      alert(ethers)
    }
  }
}
</script>
<style scoped>

</style>
