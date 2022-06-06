<script setup>
  import EasyCalendar from './components/EasyCalendar.vue'
  import { ref, onMounted, reactive, computed } from 'vue'
  const calendar = ref(null)
  const time = reactive({
    year: new Date().getFullYear(),
    month: new Date().getMonth(), // 从零开始计数0~11 
  })
  // 周标志时间。用于记录周视图下的几月几日到几月几日
  const weekFlagTime = ref(new Date().getTime())
  const weekOrMonthFlag = ref('month')

  // 几月几日
  const pWeekdays = ref([[], [], [], [], [], [], []])

  // 将时间设置为今天
  let setToday = () => {
    calendar?.value.setToday()
  }
  // 上周或者上月
  const leftArrow = () => {
    calendar?.value.leftArrow()
  }
  // 下周或者下个月
  const rightArrow = () => {
    calendar?.value.rightArrow()
  }

</script>

<template>
  <div style="width:1050px;height: 100px;position: relative;">
    <img src="./assets/calendar-bg.png" style="width:100%;height: 100%;object-fit: cover;" />
    <div
      style="position: absolute;top:0;right:0;bottom:0;left:40px;z-index: 1;display: flex;align-items: center;font-size: 30px;">
      活动日历
    </div>
  </div>
  <button @click="setToday">今天</button>
  <button @click="leftArrow">&lt;</button>
  <button @click="rightArrow">&gt;</button>
  <span v-if="weekOrMonthFlag==='month'">{{time.year}}年{{time.month+1}}月</span>
  <span v-else>{{pWeekdays[0]?.month + 1}}月{{pWeekdays[0]?.monthDay}}日 - {{pWeekdays[6]?.month +
    1}}月{{pWeekdays[6]?.monthDay}}</span>
  <button @click="weekOrMonthFlag='week'">周视图</button>
  <button @click="weekOrMonthFlag='month'">月视图</button>
  <!-- +++++++ -->
  <easy-calendar ref="calendar" v-model:time="time" v-model:weekFlagTime="weekFlagTime"
    :weekOrMonthFlag="weekOrMonthFlag" v-model:pWeekdays="pWeekdays"></easy-calendar>
  <!-- +++++++ -->
</template>

<style>
  html,
  body {
    padding: 0;
    margin: 0;
  }
</style>