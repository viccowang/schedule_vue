<template>
  <input
  type="text"
  ref="rawTimeInputRef"
  placeholder="00:00"
  v-model="timeModel"
  @focus="onFocus"
  @keyup="onKeyup"
  @blur="onBlur">
</template>
<script>
/*
 * Raw Time Input
 * Author: Vicco Wang
 * Date: 2018.03.21
 *
 * usage:
 *
 * props:
 * :data     Time时间的数据, 格式 00:00 支持.sync同步
 *
 * Event:
 * @onfocus         获得焦点时触发
 * @onKeyupSuccess  按键并验证成功时触发
 * @onBlurSuccess   失焦并验证成功是触发
 *
*/

// 00:00验证
const strTimeParse = /^(?:(?:[0-2][0-3])|(?:[0-1][0-9])):[0-5][0-9]$/
// 四位时间格式验证
const timeParse = /^(((0|1)\d)|(2[0-3]))[0-5][0-9]$/

export default {
  name: 'RawTimeInput',
  props: ['data'],
  data () {
    return {
      timeModel: ''
    }
  },
  watch: {
    data (newTime) {
      this.timeModel = newTime
    },
    timeModel (newTime) {
      this.$emit('update:data', newTime)
    }
  },
  methods: {
    onFocus () {
      this.$emit('onfocus')
      // 获得物理焦点并全选
      this.$refs.rawTimeInputRef.focus()
      this.$refs.rawTimeInputRef.select()
    },
    onKeyup () {
      const time = this.timeModel
      const timeInputLen = time.length
      if (timeInputLen < 4) {
        return
      } else if (timeInputLen === 4) {
        if (!strTimeParse.test(time)) {
          this.timeModel = ''
        }
      } else if (timeInputLen > 4) {
        if (!strTimeParse.test(time)) {
          this.timeModel = this.timeModel.substring(0, 5)
        }
      }
      // 验证通过后更新页面数据,并自动触发选择下一个时刻
      if (timeParse.test(time)) {
        const replaceTime = time.replace(/(\d\d)(\d\d)/, '$1:$2')
        // 更新时刻页面数据
        this.timeModel = replaceTime
        // 触发父组件事件
        this.$emit('onKeyupSuccess', time)
      }
    },
    onBlur () {
      const time = this.timeModel
      // 失焦并验证为正确的 00:00 格式,即可更新源数据对象
      if (strTimeParse.test(time)) {
        // 触发父组件事件
        this.$emit('onBlurSuccess', time)
      } else {
        this.timeModel = ''
      }
    }
  }
}
</script>
