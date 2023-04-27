<template>
  <div>
    <div class="header">
      画布页面
    </div>
    <div>
      <MyCanvas
        :bgImg="bgImg"
      ></MyCanvas>
    </div>
  </div>
</template>

<script>
import MyCanvas from '../components/myCanvas.vue'
export default {
  components: {
    MyCanvas
  },

  data() {
    return {
      bgImg: {}
    }
  },

  created() {
    this.bgImg = {
      width: 300,
      height: 700,
      src: 'https://img1.baidu.com/it/u=567782244,1695500002&fm=253&fmt=auto&app=138&f=JPEG?w=753&h=500'
    }
  }
}
</script>

<style lang="less" scoped>

</style>