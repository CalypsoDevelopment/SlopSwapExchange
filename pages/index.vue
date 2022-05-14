<template>
  <b-container fluid class="bg">
    <TopNavigationBarSimple />
    <b-container>
      <b-row>
        <b-col cols="12">
          <SlopSwapTrader />
          <BSCPairsApp />
        </b-col>
      </b-row>
    </b-container>
  </b-container>
</template>

<script>
import TopNavigationBarSimple from '~/components/TopNavigationBarSimple.vue'
import SlopSwapTrader from '~/components/SlopSwapTrader.vue'
import BSCPairsApp from '~/components/BSCPairsApp.vue'

export default {
  name: 'IndexPage',
  components: { SlopSwapTrader, TopNavigationBarSimple, BSCPairsApp }
}
</script>
<style scoped>
.bg {
  background-color: #e0e0e0;
  min-height: 100vh;
}
</style>
