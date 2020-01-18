<template>
  <div id="app">
    <m-header></m-header>
    <tab></tab>
    <!-- keepalive可以把已经渲染过的dom缓存到内存中 这样就不用在recommend那边多次渲染 -->
    <keep-alive>
      <router-view></router-view>
    </keep-alive>
  </div>
</template>

<script>
import MHeader from 'components/m-header/m-header'
import Tab from 'components/tab/tab'

export default {
  components: {
    MHeader,
    Tab
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable";
  #app
    color:$color-theme
</style>
