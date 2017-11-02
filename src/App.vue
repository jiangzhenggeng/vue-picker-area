<template>
  <div id="app">
    <label class="order__item-input-warp" @click="show_select_area = true">
      <input
        style="font-size: 36px"
        type="text"
        class="input__text"
        :value="area | areaFilter"
        placeholder="详细地址" disabled
      >
      <picker-area :show.sync="show_select_area" @ok="areaSelect"></picker-area>
    </label>
  </div>
</template>

<script>

  import PickerArea from './picker-area.vue'

  export default {
    name: 'app',
    data () {
      return {
        show_select_area: false,
        area: {
          province: '',
          city: '',
          county: ''
        }
      }
    },
    components: {
      PickerArea
    },
    methods: {
      areaSelect (array) {
        this.area.province = array[0] && array[0].name
        this.area.city = array[1] && array[1].name
        this.area.county = array[2] && array[2].name
      }
    },
    filters: {
      areaFilter (area) {

        return area.province ? area.province + '/' + area.city + (area.county ? '/' + area.county : '') : ''
      }
    }
  }
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
