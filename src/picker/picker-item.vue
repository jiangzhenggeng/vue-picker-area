<style lang="scss" scoped="">
  $color-background: #fff;
  $color-checked: #2c97f1;
  $color-text-main: #333;
  $color-text-second: #a8a8a8;
  .pd {
    &-select {
      &-mask {
        background: linear-gradient(to top, rgba(255, 255, 255, 0.7), rgba(255, 255, 255, 1));
        position: absolute;
        top: -37px;
        left: 0;
        width: 100%;
        height: 50%;
        z-index: 3;
      }
      &-down-mask {
        top: auto;
        bottom: -37px;
        background: linear-gradient(to bottom, rgba(255, 255, 255, 0.7), rgba(255, 255, 255, 1));
      }
      &-item {
        overflow: hidden;
        width: 100%;
        text-align: center;
        height: 440px;
        background: $color-background;
        position: relative;
        backface-visibility: hidden;
        top: 0;
        color: $color-text-second;
      }
      &-ul {
        position: relative;
      }
      &-list {
        position: absolute;
        left: 0;
        right: 0;
        z-index: 2;
        height: 68px;
        border-top: 1px solid #ccc;
        border-bottom: 1px solid #ccc;
        top: 50%;
        transform: translateY(-50%);
      }
      &-list-item {
        text-shadow: 0 1px 1px rgba(102, 102, 102, 0.6);
      }
      &-list-item {
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        line-height: 34px * 2;
        font-size: 18px * 2;
        color: $color-text-main;
      }
    }
  }
</style>

<template>
  <div class="pd-select-item">
    <div class="pd-select-mask pd-select-up-mask"></div>
    <div class="pd-select-list">
      <ul class="pd-select-ul" ref="list">
        <li class="pd-select-list-item" v-for="item in listData " :key="item.id">
          {{item.name}}
        </li>
      </ul>
    </div>
    <div class="pd-select-mask pd-select-down-mask"></div>
  </div>
</template>
<script>

  export default {
    name: 'pdSelectItem',
    data () {
      return {
        finger: {
          startY: 0,
          lastY: 0,
          startTime: 0,
          lastTime: 0,
          transformY: 0
        },
        pixel: 2
      }
    },
    props: {
      listData: {
        type: Array,
        required: true
      },
      type: {
        type: String,
        default: 'line'
      },
      value: {}
    },
    watch: {
      value (newVal) {
        let newIndex = this.listData.indexOf(newVal)
        if (newIndex > -1) {
          var h = 34 * this.pixel
          newIndex = Math.round(newIndex * h / h, 0) * h
          this.setListTransform(-newIndex, 'end')
        }
      },
      listData () {
        this.setListTransform()
        this.getPickValue(0)
      }
    },
    mounted () {
      /* 事件绑定 */
      this.$el.addEventListener('touchstart', this.itemTouchStart)
      this.$el.addEventListener('touchmove', this.itemTouchMove)
      this.$el.addEventListener('touchend', this.itemTouchEnd)
      /* 初始化状态 */
      let index = this.listData.indexOf(this.value)
      if (index === -1) {
        this.setListTransform()
        this.getPickValue(0)
      } else {
        let move = index * 34 * this.pixel
        /* 因为往上滑动所以是负 */
        this.setStyle(-move)
        this.setListTransform(-move)
      }
    },
    methods: {
      setListTransform (translateY = 0, type, time = 1000) {
        var list = this.$refs.list
        var style = list.style
        if (type === 'end') {
          style.webkitTransition = `transform ${time}ms cubic-bezier(0.19, 1, 0.22, 1)`
          style.webkitTransform = `translateY(${translateY}px)`
          list.setAttribute('scroll', translateY)
        } else {
          style.webkitTransition = ''
          style.webkitTransform = `translateY(${translateY}px)`
          list.setAttribute('scroll', translateY)
        }
      },
      itemTouchStart (event) {
        let finger = event.changedTouches[0]
        this.finger.startY = finger.pageY
        this.finger.startTime = event.timestamp || Date.now()
        this.finger.transformY = Number(this.$refs.list.getAttribute('scroll') || '0')
        event.preventDefault()
      },
      itemTouchMove (event) {
        let finger = event.changedTouches[0]
        this.finger.lastY = finger.pageY
        this.finger.lastTime = event.timestamp || Date.now()
        /* 设置css */
        let move = this.finger.lastY - this.finger.startY
        this.setStyle(move)
        event.preventDefault()
      },
      itemTouchEnd (event) {
        let finger = event.changedTouches[0]
        this.finger.lastY = finger.pageY
        this.finger.lastTime = event.timestamp || Date.now()
        let move = this.finger.lastY - this.finger.startY
        /* 计算速度 */
        /* 速度计算说明
         * 当时间小于300毫秒 最后的移动距离等于 move + 减速运动距离
         * */
        let time = this.finger.lastTime - this.finger.startTime
        let v = move / time
        /* 减速加速度a */
        let a = 1.6
        /* 设置css */
        if (time <= 300) {
          move = v * a * time
          time = 1000 + time * a
          this.setStyle(move, 'end', time)
        } else {
          this.setStyle(move, 'end')
        }
      },
      /* 设置css */
      setStyle (move, type, time) {

        const singleHeight = 34 * this.pixel
        let updateMove = move + this.finger.transformY
        /* 根据滚轮类型 line or cycle 判断 updateMove最大距离 */
        if (updateMove > 0) {
          updateMove = 0
        }
        if (updateMove < -(this.listData.length - 1) * singleHeight) {
          updateMove = -(this.listData.length - 1) * singleHeight
        }

        if (type === 'end') {
          var h = 34 * this.pixel
          updateMove = Math.round(updateMove / h, 0) * h
          this.setListTransform(updateMove, type, time)
          /* 设置$emit 延迟 */
          setTimeout(() => this.getPickValue(updateMove), 500)
        } else {
          this.setListTransform(updateMove)
        }
      },
      /* 获取选中值 */
      getPickValue (move) {
        let index = Math.abs(move / (34 * this.pixel))
        this.$emit('input', this.listData[index])
      }
    },
    beforeDestroy () {
      this.$el.removeEventListener('touchstart', this.itemTouchStart)
      this.$el.removeEventListener('touchmove', this.itemTouchMove)
      this.$el.removeEventListener('touchend', this.itemTouchEnd)
    }
  }
</script>
