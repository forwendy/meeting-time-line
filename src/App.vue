<template>
  <div id="app">
    <TimeLine :start="open_start" :end="open_end" :minTime="min" :order="reserveList" @change="change"></TimeLine>
  </div>
</template>

<script>
import TimeLine from './components/TimeLine.vue'

export default {
  name: 'App',
  components: {
    TimeLine
  },
  data() {
    return {
      min: 1,
      booking_time: 2,
      open_start: 0.5,
      open_end: 22.5,
      reserveList: [
        {
          use_end_num: 14.5,
          use_start_num: 13,
          use_start: '13:00:00',
          use_end: '14:30:00',
          id: 'feae73e8-682b-4610-a2cb-f10af022fd0f',
          avatar: 'http://test.ovuwork.com/park-os/files/circle/9bb4bfcb-8af9-4409-b301-7b632f52a822.jpg',
          remarks: '',
          reserve_name: '王炸王炸王炸王炸',
          use_date: '2021-11-29',
          status: 0 // 0未使用 1使用中 2已结束
        },
        {
          use_end_num: 18,
          use_start_num: 17.5,
          use_start: '17:30:00',
          use_end: '18:00:00',
          id: 'feae73e8-682b-4610-a2cb-f10af022fd0f',
          avatar: 'http://test.ovuwork.com/park-os/files/circle/9bb4bfcb-8af9-4409-b301-7b632f52a822.jpg',
          remarks: '',
          reserve_name: '王炸',
          use_date: '2021-11-29',
          status: 0 // 0未使用 1使用中 2已结束
        }
      ]
    }
  },
  methods: {
    change(val){
      console.log(val)
    }
  }
}
</script>

<style>
body {
  margin: 0;
}
.btn{
  display: inline-block;
  width: 100px;
  height: 50px;
  line-height: 50px;
  text-align: center;
}
</style>
