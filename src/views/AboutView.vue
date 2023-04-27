<template>
  <div>
    <div class="header">
      画布页面
    </div>
    <div>
      <MyCanvas></MyCanvas>
    </div>
  </div>
</template>

<script>
import MyCanvas from '@/components/myCanvas.vue'
export default {
  components: {
    MyCanvas
  },

  data() {
    return {
      
    }
  },

  mounted() {

  }
}
</script>

<style lang="less" scoped>

</style>