<template>
  <div class="time-input">
    <raw-time-input
      ref="timeInput"
      v-bind:class="{focused: focused}"
      v-bind:style="{height: inputHeight + 'px'}"
      :data.sync="timeModel"
      @onfocus="onFocused"
      @onKeyupSuccess="onKeyup"
      @onBlurSuccess="onBlur"
    />
  </div>
</template>
<script>
import RawTimeInput from '../RawTimeInput'

export default {
  name: 'TimeInput',
  props: {
    timeObject: Object,
    type: Number,
    upDown: String,
    focusTime: Object
  },
  data () {
    return {
      inputHeight: 25,
      focused: false,
      timeModel: ''
    }
  },
  mounted () {
    this.timeModel = this.type === 0 ? this.timeObject.bftDepartureTime : this.timeObject.bftReturnTime
  },
  watch: {
    timeObject: {
      handler (time) {
        this.timeModel = this.type === 0 ? this.timeObject.bftDepartureTime : this.timeObject.bftReturnTime
      },
      deep: true
    },
    // 监控父组件的focuseTime对象,然后做匹配是否当前input组件获得焦点
    focusTime (time) {
      if (time &&
          time.timeObj.bftUuid === this.timeObject.bftUuid &&
          time.type === this.type &&
          time.upDown === this.upDown) {
        this.focused = true
        // 获得物理焦点并全选
        console.log(this.$refs.timeInput)
        this.$refs.timeInput.$el.focus()
        this.$refs.timeInput.$el.select()
      } else {
        this.focused = false
      }
    }
  },
  methods: {
    onKeyup () {
      // 当前失去焦点后会自动更新原始数据
      this.focusToNext()
    },
    onBlur () {
      // 更新时刻数据至原始对象
      this.updateRawData()
    },
    onFocused () {
      this.$emit('update:focusTime', {
        timeObj: this.timeObject,
        type: this.type,
        upDown: this.upDown
      })
    },
    // 时刻的更新方法
    updateRawData () {
      this.$emit(
        'updateData',
        {
          timeObj: this.timeObject,
          updateTime: this.timeModel,
          type: this.type,
          upDown: this.upDown
        }
      )
    },
    focusToNext () {
      this.$emit('focusToNext', {
        timeObj: this.timeObject,
        type: this.type,
        upDown: this.upDown
      })
    }
  },
  components: { RawTimeInput }
}
</script>
