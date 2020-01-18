<template>
  <div class="slider" ref="slider">
    <div class="slider-group" ref="sliderGroup">
      <slot></slot>
    </div>
    <div class="dots">
      <span class="dot" v-for="(item,index) in dots" :key="index" :class="{active:currentPageIndex === index}"></span>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import { addClass } from 'common/js/dom'
import BScroll from 'better-scroll'

export default {
  data () {
    return {
      dots: [],
      currentPageIndex: 0
    }
  },
  //  props属性可以指定类型 也方便外部操作
  props: {
    loop: {
      type: Boolean,
      default: true
    },
    autoPlay: {
      type: Boolean,
      default: true
    },
    interval: {
      type: Number,
      default: 4000
    }
  },
  //  挂载的时候 初始化slider宽度 原点样式 图片样式
  mounted () {
    setTimeout(() => {
      this._setSliderWidth()
      this._initDots()
      this._initSlider()

      if (this.autoPlay) {
        this._play()
      }
    }, 20)
    //  监听窗口大小是否有改变
    window.addEventListener('resize', () => {
      if (!this.slider) {
        return
      }
      this._setSliderWidth(true)
      //  当窗口发生变化 会重新计算
      this.slider.refresh()
    })
  },
  methods: {
    //  计算slider的宽度 让所有图片在这个宽度里面
    _setSliderWidth (isResize) {
      this.children = this.$refs.sliderGroup.children

      let width = 0
      let sliderWidth = this.$refs.slider.clientWidth
      for (let i = 0; i < this.children.length; i++) {
        let child = this.children[i]
        addClass(child, 'slider-item')

        child.style.width = sliderWidth + 'px'
        width += sliderWidth
      }
      if (this.loop && !isResize) {
        width += 2 * sliderWidth
      }
      this.$refs.sliderGroup.style.width = width + 'px'
    },
    //  创建一个原点的数组
    _initDots () {
      this.dots = new Array(this.children.length)
    },
    //  设置slider的属性 可以在X轴方向滑动 Y轴不可以 snap就是轮播 snapLoop是轮播循环
    //  snapThreshold用手指滑动时页面可切换的阈值，大于这个阈值可以滑动的下一页
    _initSlider () {
      this.slider = new BScroll(this.$refs.slider, {
        scrollX: true,
        scrollY: false,
        momentum: false,
        snap: true,
        snapLoop: this.loop,
        snapThreshold: 0.3,
        snapSpeed: 400
        /**
         *  删除的原因是 better-scroll自己有这个click方法
         *  而我们引用的fastclick恰好被这个click所引用
         *  所以 阻止了默认的事件 因此删去
         *  click: true
         * */
      })
      //  因为BScroll在创建的时候会头尾克隆dom 所以需要在滚到最后一张的时候index-1
      this.slider.on('scrollEnd', () => {
        let pageIndex = this.slider.getCurrentPage().pageX
        if (this.loop) {
          pageIndex -= 1
        }
        this.currentPageIndex = pageIndex
        //  如果在快到4s的时候 手动滑动 效果非常奇怪 所以判断是否是自动播放 不是就计算index
        if (this.autoPlay) {
          clearTimeout(this.timer)
          this._play()
        }
      })
    },
    //  手动轮播
    _play () {
      let pageIndex = this.currentPageIndex + 1
      if (this.loop) {
        pageIndex += 1
      }
      this.timer = setTimeout(() => {
        this.slider.goToPage(pageIndex, 0, 400)
      }, this.interval)
    }
  },
  destroyed () {
    clearTimeout(this.timer)
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
@import '~common/stylus/variable'

.slider {
  min-height: 1px

  .slider-group {
    position: relative
    overflow: hidden
    white-space: nowrap

    .slider-item {
      float: left
      box-sizing: border-box
      overflow: hidden
      text-align: center

      a {
        display: block
        width: 100%
        overflow: hidden
        text-decoration: none
      }

      img {
        display: block
        width: 100%
      }
    }
  }

  .dots {
    position: absolute
    right: 0
    left: 0
    bottom: 12px
    text-align: center
    font-size: 0

    .dot {
      display: inline-block
      margin: 0 4px
      width: 8px
      height: 8px
      border-radius: 50%
      background: $color-text-l

      &.active {
        width: 20px
        border-radius: 5px
        background: $color-text-ll
      }
    }
  }
}
</style>
