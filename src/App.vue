<template>
  <div id="scheduleApp">
      <div class="loading" v-if="loading">Loading...</div>
      <toolbar
        @save="save"/>
      <div class="schedule-main" v-if="!loading">
        <div class="schedule-section-left">
            <div class="schedule-table">
              <!-- 表头 -->
              <div class="schedule-header">
                <!-- 班次头 -->
                <div class="schedule-bc-header" v-bind:style="{width: style.bcWidth +'px'}">
                    <ul>
                      <li>班次</li>
                      <li>班次类型</li>
                      <li class="higher">离站名称</li>
                    </ul>
                </div>
                <!-- 时刻头 -->
                <div class="schedule-time-header"
                    v-for="(time, index) in timeData[0]" :key="index"
                    v-bind:style="{width: style.timeWidth +'px'}">
                    <ul>
                      <li>{{ index + 1 }}</li>
                      <li>单班</li>
                      <li class="start-end-station higher">
                        <div v-bind:style="{width: style.timeWidth / 2 +'px'}">{{ rawData.bfUpStartStation}}</div>
                        <div v-if="bfType === '1'" v-bind:style="{width: style.timeWidth / 2 +'px'}">{{ rawData.bfDownStartStation}}</div>
                      </li>
                    </ul>
                </div>
              </div>
              <div class="schedule-content">
                  <div class="schedule-bc-content" v-bind:style="{width: style.bcWidth +'px'}">
                      <ul>
                        <li v-for="(time, index) in timeData" :key="index">{{ index + 1}}</li>
                      </ul>
                  </div>
                  <div class="schedule-time-content">
                      <!-- row -->
                      <div class="schedule-time-row" v-for="(row, rowIndex) in timeData" :key="rowIndex">
                        <!-- 时刻大列 -->
                        <div v-for="(timeItem, itemIdx) in row" :key="itemIdx" v-bind:style="{width: style.timeWidth +'px'}">
                          <ul>
                            <li v-bind:style="{width: style.timeWidth / 2 +'px'}">
                              <!--
                                  time-object  当前时刻数据对象
                                  type   0 -> 字段 bftDepartureTime   1 -> bftReturnTime
                                  upDwon  up -> upFt   down -> downFt
                                  focusTime   双向绑定当前全局时刻中获得焦点的时刻
                                -->
                              <!-- 时刻1 -->
                              <time-input
                                :time-object="timeItem.upFt"
                                :type="0"
                                upDown="up"
                                :focusTime.sync="focusTime"
                                @focusToNext="focusToNext"
                                @updateData="updateData"/>
                              <!-- 时刻2 -->
                              <time-input
                                :time-object="timeItem.upFt"
                                :type="1"
                                upDown="up"
                                :focusTime.sync="focusTime"
                                @focusToNext="focusToNext"
                                @updateData="updateData"/>
                            </li>
                            <li v-if="bfType === '1'" v-bind:style="{width: style.timeWidth / 2 +'px'}">
                              <!-- 时刻3 -->
                              <time-input
                                :time-object="timeItem.downFt"
                                :type="0"
                                upDown="down"
                                :focusTime.sync="focusTime"
                                @focusToNext="focusToNext"
                                @updateData="updateData" />
                              <!-- 时刻4 -->
                              <time-input
                                :time-object="timeItem.downFt"
                                :type="1"
                                upDown="down"
                                :focusTime.sync="focusTime"
                                @focusToNext="focusToNext"
                                @updateData="updateData" />
                            </li>
                          </ul>
                      </div>
                      </div>

                  </div>
              </div>
            </div>
        </div>
        <div class="schedule-section-right">
          <calc-right
            :rawData="rawData"
            :focusTime="focusTime"
            @checkedDepart="focusDepart"
            @fastEntry="fastEntry"
            ></calc-right>
        </div>
      </div>

  </div>
</template>

<script>
import axios from 'axios'
import Toolbar from './components/Toolbar'
import TimeInput from './components/TimeInput'
import CalcRight from './components/CalcRight'
import './schedule.scss'

export default {
  name: 'App',
  data () {
    return {
      // 每大列时刻包含几个时刻小列
      columnLen: 4,
      // 焦点到底部返回时是顺序选择还是夸列选择
      // stand -> 夸列, fluid 顺序
      focusMode: 'stand',
      // 表格自定义样式
      style: {
        bcWidth: 120, // 班次大列宽度
        timeWidth: 240 // 时刻大列宽度 每个时刻宽度= time_width / columnLen
      },
      focusTime: null, // 当前获得焦点的时刻数据对象
      bfType: null,
      rawData: {},
      // 时刻数据
      timeData: {},
      loading: true
    }
  },
  watch: {
  },
  mounted () {
    this.initData()
  },
  methods: {
    initData () {
      axios.post(
        // get Uri
        'http://192.168.10.53:8080/scp_xt/fahrplan/showTable1',
        // get params
        {
          id: '0788b529bf694821b7cd'
        }
      ).then(res => {
        // 原始全部数据
        this.rawData = res.data.data
        // 对应时刻数据
        this.timeData = res.data.data.fahrplanTimeList
        //
        this.style.timeWidth = this.rawData.bfType === '1' ? 240 : 120

        //
        this.bfType = this.rawData.bfType
        //
        this.loading = false
        //
        console.log(this.rawData)
      })
    },
    updateData (timeModel) {
      const data = this.rawData.fahrplanTimeList
      /*
      * timeObj 需要更新的原始时刻对象
      * updateTime 最新的时刻
      * type  0 -> bftDepartureTime   1 -> returnTime
      * upDown  up -> upFt    down -> downFt
      */
      const { timeObj, updateTime, type, upDown } = timeModel

      // 遍历原始数据并更新对应的时刻
      for (let i = 0, l = data.length; i < l; i++) {
        let rowData = data[i]
        for (let j = 0, jl = rowData.length; j < jl; j++) {
          let timeItem = upDown === 'up' ? rowData[j].upFt : rowData[j].downFt
          if (timeObj.bftUuid === timeItem.bftUuid) {
            const key = type === 0 ? 'bftDepartureTime' : 'bftReturnTime'
            timeItem[key] = updateTime
            break
          }
        }
      }
      // 更新原始数据
      this.rawData.fahrplanTimeList = data
    },
    // 自动选择下一个
    focusToNext (timeModel) {
      const data = this.rawData.fahrplanTimeList
      let { timeObj, type, upDown } = timeModel

      // 遍历原始数据并更新对应的时刻
      for (let i = 0, rowLen = data.length; i < rowLen; i++) {
        let rowIndex = i // 行下标
        let rowData = data[rowIndex]
        for (let j = 0, jl = rowData.length; j < jl; j++) {
          let itemIndex = j // 列下标
          let timeItem = this.bfType === '1' ? upDown === 'up' ? rowData[itemIndex].upFt : rowData[itemIndex].downFt : rowData[itemIndex].upFt
          if (timeObj.bftUuid === timeItem.bftUuid) {
            let timeObj = null

            // 不是最后一行(不是最后一个班次),则直接向下选
            if (rowIndex < rowLen - 1) {
              timeObj = this.bfType === '1' ? upDown === 'up' ? data[rowIndex + 1][j].upFt : data[rowIndex + 1][j].downFt : data[rowIndex + 1][j].upFt
            // 如果是最后一行(最后一个班次),则需要跳转到下一行
            } else {
              // 遇到跳列将自动转入下一列
              if (this.focusMode === 'fluid') {
                // 如果当前选择的是左侧列,则直接返回右侧列第一个
                if (type === 0) {
                  timeObj = this.bfType === '1' ? upDown === 'up' ? data[0][j].upFt : data[0][j].downFt : data[0][j].upFt
                  type = 1
                // 如果当前选择的是右侧列,则返回下一大列数据的左侧列第一个数据
                } else {
                  // 如果有下一大列
                  if (itemIndex < jl - 1) {
                    timeObj = this.bfType === '1' ? upDown === 'up' ? data[0][j].downFt : data[0][j + 1].upFt : data[0][j + 1].upFt
                    type = 0
                    upDown = this.bfType === '1' ? upDown === 'up' ? 'down' : 'up' : 'up'
                  } else {
                    // console.log('最后')
                    // 如果没有下一大列时刻,则移除当前的focusTime
                    // this.focusTime = null
                    return
                  }
                }
              // 遇到跳列
              } else if (this.focusMode === 'stand') {
                // 如果当前选择的是左侧列,则直接跳到下一个大列左侧列
                // 如果有下一大列
                if (itemIndex < jl - 1) {
                  timeObj = this.bfType === '1' ? upDown === 'up' ? data[0][j + 1].upFt : data[0][j + 1].downFt : data[0][j + 1].upFt
                } else {
                  // this.focusTime = null
                  return
                }
              }
            }

            const focusTime = { timeObj, type, upDown }
            this.focusTime = focusTime
            break
          }
        }
      }
    },
    // 主副站切换
    focusDepart (checked) {
      const data = this.timeData
      const { timeObj, type, upDown } = this.focusTime
      for (let i = 0, l = data.length; i < l; i++) {
        let rowData = data[i]
        for (let j = 0, jl = rowData.length; j < jl; j++) {
          let timeItem = upDown === 'up' ? rowData[j].upFt : rowData[j].downFt
          if (timeObj.bftUuid === timeItem.bftUuid) {
            // 更新焦点时刻为对应的时刻信息
            if (checked === '0' && upDown !== 'up') {
              this.focusTime = {
                timeObj: rowData[j].upFt,
                type,
                upDown: 'up'
              }
            } else if (checked === '1' && upDown !== 'down') {
              this.focusTime = {
                timeObj: rowData[j].downFt,
                type,
                upDown: 'down'
              }
            }
            break
          }
        }
      }
    },
    // 快速录入
    fastEntry () {
      console.log('fast entry event')
    },
    save () {
      console.log(this.rawData)
    }
  },
  components: { Toolbar, TimeInput, CalcRight }
}
</script>

<style lang="scss">

body,html {
  padding:0;
  margin:0;
  width:100%;
  height:100%;
  font-size: 9px;

  ul, li {
    padding:0;margin:0;
    list-style:none;
  }

  input,textarea, button { outline: none; }

}

</style>
