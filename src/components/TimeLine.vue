<template>
  <div class="time-line-group">
    <div v-scroll class="time-line-wrapper">
      <div class="time-line-tick">
        <div class="time-line-tick-item" v-for="obj in timeTick" :key="obj.time">
          <p>{{ obj.show }}</p>
        </div>
      </div>
      <!-- 时间轴 -->
      <div class="time-line">
        <div class="time-line-item" v-for="(obj, i) in timeline" :key="i" :data-time="obj.time" :class="{ disabled: obj.disabled }" @click="select(obj, i)"></div>
        <!-- 预定记录 -->
        <div class="order-item" v-for="obj in orderList" :key="obj.time" :style="{ left: obj.left + 'px', width: obj.width + 'px' }">
          <img :src="obj.avatar" />
          <p v-if="obj.cell > 1">{{ obj.reserve_name }}</p>
        </div>
        <!-- 滑块 -->
        <div v-show="orderWidth" v-move class="slide-block" :style="{ left: orderLeft + 'px', width: orderWidth + 'px' }">
          <div v-drag class="btn">
            <img src="@/assets/images/meeting-slide-icon@2x.png" />
          </div>
        </div>
      </div>
    </div>
    <div v-if="orderLeft" class="control-bar">
      <div class="control-bar-btn" @click="minusTime">-</div>
      <div class="text">
        <div class="long">共{{ orderEnd - orderStart }}小时</div>
        <div class="time">{{ startTime }} - {{ endTime }}</div>
      </div>
      <div class="control-bar-btn" @click="plusTime">+</div>
    </div>
  </div>
</template>

<script>
export default {
  created() {
    this.initTimeLine()
    this.initOrder()
  },
  data() {
    return {
      // 时间刻度标识 - 整点
      timeTick: [],
      // 时间单元格列表
      timeline: [],
      // 预定记录
      orderList: [],
      // time-line-wrapper 左边的距离
      wrapLeft: 0,
      // 单元格宽度
      width: 32,
      orderLeft: 0,
      // 预定开始时间
      orderStart: 0,
      // 预定结束时间
      orderEnd: 0,
      // 最小单元格数
      min: 0,
      // 最大单元格数
      max: 0,
      // 靠左距离范围
      leftMin: 0,
      leftMax: 0,
      orderStartIndex: 0
    }
  },
  computed: {
    orderWidth() {
      return (this.orderEnd - this.orderStart) * this.width * 2
    },
    startTime() {
      return this.format(this.orderStart)
    },
    endTime() {
      return this.format(this.orderEnd)
    }
  },
  methods: {
    // 初始化会议室时间轴
    initTimeLine() {
      const timeline = []
      for (var i = 0; i < 24; i++) {
        // 会议室开放时间
        const point = i < this.start || i >= this.end
        const half = i + 0.5 < this.start || i + 0.5 >= this.end
        timeline.push({
          time: i,
          show: i > 9 ? i + ':00' : '0' + i + ':00',
          isHalf: false,
          disabled: point
        })
        timeline.push({
          time: i + 0.5,
          show: i > 9 ? i + ':30' : '0' + i + ':30',
          isHalf: true,
          disabled: half
        })
      }
      this.timeTick = timeline.filter((el, i) => {
        return i % 2 === 0
      })
      this.timeline = timeline
      // 是否当天
      if (this.today) {
        const hour = new Date().getHours()
        const minute = new Date().getMinutes() > 30
        this.timeline = this.timeline.map((el) => {
          if (el.time < hour) {
            el.disabled = true
          }
          if (el.time === hour && minute) {
            el.disabled = true
          }
          return el
        })
      }
      // 计算第一个可以定时间
      const start = this.timeline.findIndex((el) => {
        return !el.disabled
      })
      this.wrapLeft = start * this.width
    },
    // 初始化预定记录数据
    initOrder() {
      const arr = []
      for (let i = 0; i < this.order.length; i++) {
        const order = this.order[i]
        const long = order.use_end_num - order.use_start_num
        const startIndex = this.timeline.findIndex((el) => {
          return el.time === order.use_start_num
        })
        arr.push({
          left: this.width * startIndex,
          width: this.width * long * 2,
          cell: long * 2,
          ...order
        })
        // 时间轴上标志
        this.timeline = this.timeline.map((el) => {
          if (el.time >= order.use_start_num && el.time < order.use_start_num + long) {
            el.disabled = true
          }
          return el
        })
      }
      this.orderList = arr
    },
    // 点选时间轴
    select(obj, i) {
      if (obj.disabled) return
      const left = i * this.width
      this.orderStartIndex = i
      this.orderStart = obj.time
      this.orderEnd = obj.time + this.minTime
      // 计算可增减区域
      this.verify(i)
      // 计算可移动区域
      this.calcRange(i)
      if (left <= this.leftMax) {
        this.orderLeft = left
      } else {
        alert('时长超出')
      }
      this.change()
    },
    // 计算可增减区域
    verify(i) {
      const min = this.minTime * 2
      let max = this.maxTime * 2
      let timeline = this.timeline.slice(i, this.timeline.length)
      const disabledIndex = timeline.findIndex((el) => {
        return el.disabled
      })
      max = max < disabledIndex ? max : disabledIndex
      this.min = min
      this.max = max
      console.log(this.min, this.max)
    },
    // 计算可移动区域
    calcRange(i) {
      const long = (this.orderEnd - this.orderStart) * 2
      let prevArr = this.timeline.slice(0, i).reverse()
      let nextArr = this.timeline.slice(i, this.timeline.length)
      const prevArrIndex = prevArr.findIndex((el) => {
        return el.disabled
      })
      const nextArrIndex = nextArr.findIndex((el) => {
        return el.disabled
      })
      prevArr = prevArr.slice(0, prevArrIndex).reverse()
      nextArr = nextArr.slice(0, nextArrIndex)
      let range = [].concat(prevArr, nextArr)
      range = range.slice(0, range.length - long + 1)
      // console.log(range)
      this.leftMin = range[0].time * 2 * this.width
      this.leftMax = range[range.length - 1].time * 2 * this.width
      // console.log(this.leftMin, this.leftMax)
    },
    // 加0.5时间点
    plusTime() {
      const long = (this.orderEnd - this.orderStart) * 2
      if (long < this.max) {
        this.orderEnd = this.orderEnd + 0.5
      } else {
        alert('已到最大值')
      }
      // 计算可移动区域
      this.calcRange(this.orderStartIndex)
      this.change()
    },
    // 减0.5时间点
    minusTime() {
      const long = (this.orderEnd - this.orderStart) * 2
      if (long > this.min) {
        this.orderEnd = this.orderEnd - 0.5
      } else {
        alert('已到最小值')
      }
      // 计算可移动区域
      this.calcRange(this.orderStartIndex)
      this.change()
    },
    format(timeNum) {
      const hour = String(timeNum).split('.')[0]
      const minute = String(timeNum).split('.')[1]
      if (minute) {
        return hour > 9 ? hour + ':30' : '0' + hour + ':30'
      } else {
        return hour > 9 ? hour + ':00' : '0' + hour + ':00'
      }
    },
    change() {
      this.$emit('change', {
        orderStart: this.orderStart,
        orderEnd: this.orderEnd
      })
    }
  },
  directives: {
    scroll: {
      inserted: function (el, binding, vnode) {
        const vm = vnode.context
        el.scrollLeft = vm.wrapLeft
      }
    },
    move: {
      inserted: function (el, bind, vnode) {
        const vm = vnode.context
        el.ontouchstart = (e) => {
          const disx = e.targetTouches[0].pageX - el.offsetLeft
          // const orderLeft = vm.orderLeft
          document.ontouchmove = function (e) {
            let X = e.targetTouches[0].pageX - disx
            vm.orderLeft = X
          }
          document.ontouchend = function (e) {
            const long = Math.round(vm.orderLeft / vm.width)
            let X = long * vm.width
            if (X > vm.leftMax) {
              // 不可预订
              vm.orderLeft = vm.leftMax
            } else if (X < vm.leftMin) {
              // 不可预订
              vm.orderLeft = vm.leftMin
            } else {
              // 可以预定
              vm.orderLeft = X
            }
            vm.orderStartIndex = vm.orderLeft / vm.width
            const obj = vm.timeline[vm.orderStartIndex]
            const timeLong = vm.orderEnd - vm.orderStart
            vm.orderStart = obj.time
            vm.orderEnd = obj.time + timeLong
            vm.verify(vm.orderStartIndex)
            vm.change()
            document.ontouchmove = document.ontouchend = null
            e.preventDefault()
            e.stopPropagation()
          }
          e.preventDefault()
          e.stopPropagation()
        }
      }
    },
    drag: {
      inserted: function (el, bind, vnode) {
        const vm = vnode.context
        el.ontouchstart = (e) => {
          const disx = e.targetTouches[0].pageX
          const width = vm.orderWidth
          let endWidth
          document.ontouchmove = function (e) {
            let X = e.targetTouches[0].pageX - disx
            endWidth = width + X
            if (endWidth < vm.min * vm.width) {
              endWidth = vm.min * vm.width
            }
            if (endWidth > vm.max * vm.width) {
              endWidth = vm.max * vm.width
            }
            el.parentNode.style.width = endWidth + 'px'
          }
          document.ontouchend = function (e) {
            const time = Math.round(endWidth / vm.width)
            el.parentNode.style.width = time * vm.width + 'px'
            vm.orderEnd = vm.orderStart + time / 2
            vm.calcRange(vm.orderStartIndex)
            vm.change()
            document.ontouchmove = document.ontouchend = null
            e.preventDefault()
            e.stopPropagation()
          }
          e.preventDefault()
          e.stopPropagation()
        }
      }
    }
  },
  props: {
    // 会议室开始时间
    start: {
      type: Number,
      default() {
        return 0
      }
    },
    // 会议室结束时间
    end: {
      type: Number,
      default() {
        return 23
      }
    },
    // 最大时间限制 默认没有
    maxTime: {
      type: Number,
      default() {
        return 24
      }
    },
    // 最小时间 默认为0.5 （半小时）
    minTime: {
      type: Number,
      default() {
        return 0.5
      }
    },
    // 是否为当天
    today: {
      type: Boolean,
      default() {
        return true
      }
    },
    // 预定记录
    order: {
      type: Array,
      default() {
        return []
      }
    }
  }
}
</script>

<style scoped>
.time-line-group {
  margin: 50px 10px;
}
.time-line-wrapper {
  overflow-x: scroll;
  position: relative;
}
.time-line-wrapper::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 1px;
  height: 100%;
  background: #dddddd;
  z-index: 2;
}
.time-line,
.time-line-tick {
  height: 30px;
  display: flex;
  position: relative;
}
.time-line-item,
.time-line-tick-item {
  width: 64px;
  flex-shrink: 0;
  position: relative;
}
.time-line-tick-item::before {
  content: '';
  position: absolute;
  right: 0;
  top: 0;
  bottom: 0;
  width: 1px;
  height: 100%;
  background: #dddddd;
}
.time-line-item::after,
.time-line-tick-item::after {
  content: '';
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  width: 100%;
  height: 1px;
  background: #dddddd;
}
.time-line-tick-item {
  height: 30px;
  position: relative;
}
.time-line-tick-item p {
  font-size: 12px;
  color: #333333;
  line-height: 30px;
  padding: 0 3px;
  box-sizing: border-box;
  margin: 0;
}
.time-line-tick-item p::before {
  content: '';
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  width: 100%;
  height: 1px;
  background: #dddddd;
}
.time-line-item {
  width: 32px;
}
.time-line-item:before {
  content: '';
  position: absolute;
  right: 0;
  top: auto;
  bottom: 0;
  width: 1px;
  height: 50%;
  background: #dddddd;
}
.time-line-item:nth-child(2n):before {
  top: 0;
  height: 100%;
}
.time-line-item.disabled {
  background: #f2f2f2;
}
.order-item {
  height: 29px;
  padding: 0 3px;
  box-sizing: border-box;
  position: absolute;
  background: #dddddd;
  color: #333333;
  font-size: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.order-item img {
  width: 20px;
  height: 20px;
}
.order-item p {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  -o-text-overflow: ellipsis;
  margin: 0;
  margin-left: 5px;
}
.slide-block {
  position: absolute;
  height: 29px;
  background: #0b7aff;
}
.slide-block .btn {
  position: absolute;
  right: -6px;
  top: 0;
  bottom: 0;
  margin: auto;
  height: 100%;
  width: 13px;
  display: flex;
  align-items: center;
}
.slide-block .btn img {
  width: 13px;
  height: 13px;
}
.control-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 50px;
}
.control-bar .text {
  font-size: 15px;
  text-align: center;
}
.control-bar .time {
  font-size: 12px;
}
.control-bar-btn {
  width: 21px;
  height: 21px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid #0b7aff;
  border-radius: 50%;
  color: #0b7aff;
  line-height: 1;
}
</style>
