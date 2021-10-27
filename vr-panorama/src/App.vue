<template>
  <panorama/>
</template>

<script>
// import panorama from './components/panorama'
import panorama from './components/panorama.vue'

export default {
  name: 'App',
  components: {
    panorama
  }
}
</script>
<style>
body {
  margin: 0;
  overflow: hidden; /* 隐藏body窗口区域滚动条 */
}
</style>