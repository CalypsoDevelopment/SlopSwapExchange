<template>
  <div>
    <b-nav class="simple-top-nav">
      <b-nav-item>
        <b-button><i class="fa-solid fa-bars fa-1x" style="color: #FFFFFF;" /></b-button>
      </b-nav-item>
    </b-nav>
  </div>
</template>
<script>
export default {
  name: 'TopNavigationBarSimple',
  components: {},
  data () {
    return {
      greeting: 'Hello World!'
    }
  }
}
</script>
<style scoped>
.nav.a {
  color: #6c757d;
}
.simple-top-nav:hover {
  color: #505960;
}

</style>
