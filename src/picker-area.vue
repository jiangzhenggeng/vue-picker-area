<style lang="less">
  .pd-select-wrap {
    position: fixed;
    z-index: 8;
    background-color: rgba(0, 0, 0, 0.5);
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
  }

  .pd-select-inner {
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    background-color: #fff;
  }

  .pd-select-btn-wrap {
    display: flex;
    justify-content: space-between;
    height: 80px;
    align-items: stretch;
    font-size: 32px;
    color: #fe5341;
  }

  .pd-select-btn {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 0 24px;
  }

  .pd-select-box {
    display: flex;
    border-top: 1px solid #ccc;
    align-items: stretch;
  }

  .fade-enter-active, .fade-leave-active {
    transition: opacity .5s;
  }

  .fade-enter, .fade-leave-active {
    opacity: 0;
  }
</style>

<template>
  <transition name="fade">
    <div class="pd-select-wrap" @click.stop="close" v-if="inner_show">
      <div class="pd-select-inner">
        <div class="pd-select-btn-wrap">
          <div class="pd-select-btn" @click.stop="close">关闭</div>
          <div class="pd-select-btn" @click.stop="ok">确定</div>
        </div>
        <div class="pd-select-box">
          <picker-item v-if="provinceList.length" :listData="provinceList" v-model="select_province"></picker-item>
          <picker-item v-if="cityList.length" :listData="cityList" v-model="select_city"></picker-item>
          <picker-item v-if="countyList.length" :listData="countyList" v-model="select_county"></picker-item>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
  import PickerItem from './picker/picker-item.vue'
  import cityList from './cityList'

  function getArea (pid) {
    var areaArray = []
    cityList.forEach(item => {
      if (item.pid === pid) {
        areaArray.push(item)
      }
    })
    return areaArray
  }

  function nameToId (name) {
    var id = 0
    cityList.forEach(item => {
      if (item.name === name) {
        id = item.id
      }
    })
    return id
  }

  export default {
    name: 'pdIosSelect',
    props: {
      show: {
        type: Boolean,
        default: false
      },
      province: {
        type: String,
        default: '北京'
      },
      city: {
        type: String,
        default: '北京市'
      },
      county: {
        type: String,
        default: '东城区'
      }
    },
    data () {
      return {
        provinceList: getArea(0),
        cityList: getArea(1),
        countyList: getArea(2),
        select_province: this.province,
        select_city: this.city,
        select_county: this.county,
        inner_show: this.show
      }
    },
    watch: {
      select_province (newVal) {
        this.cityList = getArea(newVal.id)
      },
      select_city (newVal) {
        this.countyList = getArea(newVal.id)
      },
      show (newVal) {
        this.inner_show = newVal
      }
    },
    components: {
      PickerItem
    },
    methods: {
      getReturnData () {
        var array = []
        this.provinceList.length && array.push(this.select_province)
        this.cityList.length && array.push(this.select_city)
        this.countyList.length && array.push(this.select_county)

        return array
      },
      close () {
        this.inner_show = false
        this.$emit('update:show', false)
        this.$emit('close', this.getReturnData())
      },
      ok () {
        this.$emit('update:show', false)
        this.inner_show = false
        this.$emit('ok', this.getReturnData())
      }
    }
  }
</script>