<template>
    <div class="calcForm">
        <!-- 快速录入 -->
        <div class="model-right-block">
            <div class="model-title">快速录入</div>
            <div class="model-content column">
            <label class="label-item">
                <input v-model="calcData.mainSubCtx" type="checkbox">主副站联动
            </label>
                <button>计算副站车次</button>
            </div>
        </div>
        <!-- 选择主副站 -->
        <div class="model-right-block">
            <div class="model-title">选择主副站</div>
            <div class="model-content">
            <label class="label-item inline" style="margin-right:15px;">
                <input v-model="calcData.depart" type="radio" value="0">主站发车
            </label>
            <label class="label-item inline">
                <input v-model="calcData.depart" type="radio" value="1">副站发车
            </label>
            </div>
        </div>
        <!--  -->
        <div class="model-right-block">
            <div class="model-content column">
            <label class="label-item">
                发车间隔 <input class="small" type="number" v-model="calcData.dispathDistance">
            </label>
            <label class="label-item">
                发车时间 <raw-time-input class="small" type="text" :data.sync="calcData.dispathTime" />
            </label>
            <label class="label-item">
                停站时间 <input class="small" type="number" v-model="calcData.parkingTime">
            </label>
            <button @click="fastEntry">快速录入</button>
            </div>
        </div>
        <!-- 批量计算到站时间 -->
        <div class="model-right-block">
            <div class="model-title">批量计算到站时间</div>
            <div class="model-content column">
            <label class="label-item">
                单程时间 <input class="small" type="number" v-model="calcData.oneWayTime">
            </label>
            <label class="label-item">
                <input type="checkbox" disabled v-model="calcData.isSmoothTransition"> 平滑过度 <input disabled style="width:60px" type="number" v-model="calcData.smoothTime">
            </label>
            <button>计算到站时间</button>
            </div>
        </div>
        <!-- 首末时间参数 -->
         <div class="model-right-block">
            <div class="model-content column">
            <label class="label-item">
                首班时间 <raw-time-input readonly="readonly" class="small" type="text" :data.sync="calcData.firstClassTime" />
            </label>
            <label class="label-item">
                末班时间 <raw-time-input readonly="readonly" class="small" type="text" :data.sync="calcData.lastClassTime" />
            </label>
            </div>
        </div>
        <!-- 说明 -->
        <div class="model-right-block">
            This is a description...
        </div>
    </div>
</template>
<script>
import RawTimeInput from '../RawTimeInput'

export default {
  name: 'ClacRight',
  props: {
    rawData: Object,
    focusTime: Object
  },
  data () {
    return {
      calcData: {
        mainSubCtx: false,
        depart: '0',
        dispathDistance: 6, // bfUpSpaceTime
        dispathTime: '00:00',
        parkingTime: 10, // bfUpRestTime
        oneWayTime: 10, // bfUpRunTime
        isSmoothTransition: false,
        smoothTime: 0,
        firstClassTime: '00:00', // bfUpFirstHour
        lastClassTime: '00:00' // bfUpFinalHour
      }
    }
  },
  watch: {
    // 监控主副站选择
    'calcData.depart': function (checked) {
      this.checkedDepart(checked)
    },
    // 监控父组件的获得焦点的时间对象
    focusTime (time) {
      const { timeObj, type } = time
      this.calcData.dispathTime = type === 0 ? timeObj.bftDepartureTime : timeObj.bftReturnTime
    }
  },
  mounted () {
    // 根据初始数据初始化一些值
    let clearWatch = this.$watch('rawData', (data) => {
      if (data) {
        this.calcData.parkingTime = data.bfUpRestTime
        this.calcData.oneWayTime = data.bfUpRunTime
        this.calcData.dispathDistance = data.bfUpSpaceTime
        this.calcData.firstClassTime = data.bfUpFirstHour
        this.calcData.lastClassTime = data.bfUpFinalHour
        // 初始化完毕后清除当前watch
        clearWatch()
      }
    })
  },
  methods: {
    // 选择主副站
    checkedDepart (checked) {
      // 有获得焦点的时刻才触发父组件操作
      if (this.focusTime) this.$emit('checkedDepart', checked)
    },
    fastEntry () {
      const calcData = this.calcData
      this.$emit('fastEntry', calcData)
    }
  },
  components: { RawTimeInput }
}
</script>
<style lang="scss" scoped>
input, button {
    border-radius:5px;
    border: 1px solid #e0e0e0;
}
button {
    background-color:#f2f2f2;
}
</style>
