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

// 活动
const activities = reactive([])
// 开始日期
const startDate = ref(null)
// 结束日期
const endDate = ref(null)
// 活动名称
const activityName = ref(null)
// 背景颜色
const bgColor = ref('#fff4f6')
// 文字颜色
const textColor = ref('#000000')
// 添加活动
const addActivity = () => {
  console.log(bgColor.value, 'bgColor');
  const a = startDate.value.split('-')
  const aa = new Date(a[0], a[1] - 1, a[2], 0, 0, 0, 0)
  const b = endDate.value.split('-')
  const bb = new Date(b[0], b[1] - 1, b[2], 0, 0, 0, 0)
  if (aa > bb) {
    return alert('开始日期不能大于结束日期')
  }
  activities.push({
    title: activityName.value,
    beginTime: aa,
    endTime: bb,
    bgColor: bgColor.value,
    textColor:textColor.value
  })
}
</script>

<template>
  <div style="display:flex;align-items: stretch;height: 1000px;">
    <div style="flex:0 0 auto;width:211px;padding: 30px;">
      <button
        :class="weekOrMonthFlag === 'week' ? 'active' : ''"
        @click="weekOrMonthFlag = 'week'"
      >周视图</button>
      &nbsp;&nbsp;
      <button
        :class="weekOrMonthFlag === 'month' ? 'active' : ''"
        @click="weekOrMonthFlag = 'month'"
      >月视图</button>
      <br />
      <br />
      <button class="today" v-if="weekOrMonthFlag === 'month'" @click="setToday">今天</button>
      <br />
      <br />
      <button class="today" @click="leftArrow">{{ weekOrMonthFlag === 'week' ? '上一周' : '上一月' }}</button>
      &nbsp;&nbsp;
      <button
        class="today"
        @click="rightArrow"
      >{{ weekOrMonthFlag === 'week' ? '下一周' : '下一月' }}</button>

      <br />
      <br />
      <span v-if="weekOrMonthFlag === 'month'">{{ time.year }}年{{ time.month + 1 }}月</span>
      <span v-else>
        {{ pWeekdays[0]?.month + 1 }}月{{ pWeekdays[0]?.monthDay }}日 - {{
          pWeekdays[6]?.month +
            1
        }}月{{ pWeekdays[6]?.monthDay }}
      </span>
      <br />
      <br />
      <div style="border:1px solid #ccc;padding:10px">
        <h4>添加活动</h4>
        <div>开始日期</div>
        <input type="date" v-model="startDate" />
        <br />
        <br />

        <div>结束日期</div>
        <input type="date" v-model="endDate" />
        <br />
        <br />

        <div>活动名称</div>
        <input style="width:100px" type="text" v-model="activityName" />
        <br />
        <br />背景颜色
        <input style="width:100px" type="input" v-model="bgColor" />
        <br />
        <br />文字颜色
        <input style="width:100px" type="input" v-model="textColor" />
        <button @click="addActivity">添加</button>
      </div>
    </div>
    <div>
      <div style="width:1050px;height: 100px;position: relative;margin-bottom: 20px;">
        <img src="./assets/calendar-bg.png" style="width:100%;height: 100%;object-fit: cover;" />
        <div
          style="position: absolute;top:0;right:0;bottom:0;left:40px;z-index: 1;display: flex;align-items: center;font-size: 30px;"
        >活动日历</div>
      </div>
      <!-- +++++++ -->
      <easy-calendar
        ref="calendar"
        v-model:time="time"
        v-model:weekFlagTime="weekFlagTime"
        :weekOrMonthFlag="weekOrMonthFlag"
        v-model:pWeekdays="pWeekdays"
        :activities="activities"
      ></easy-calendar>
      <!-- +++++++ -->
    </div>
  </div>
</template>

<style>
html,
body {
  padding: 0;
  margin: 0;
}
div {
  box-sizing: border-box;
}
.today {
}

button.active {
  background-color: orange;
}
</style>