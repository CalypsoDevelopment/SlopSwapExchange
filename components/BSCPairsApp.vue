<template>
  <div />
</template>
<script>

export default {
  name: 'BSCPairsApp',
  components: {},
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

  }
}
</script>
<style scoped>

</style>
