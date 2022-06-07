<script setup>
import { ref, computed, onMounted, toRaw } from 'vue'
const props = defineProps({
  time: {
    type: Object,
    default() {
      return {
        year: new Date().getFullYear(),
        month: new Date().getMonth(), // 从零开始计数0~11 
      }
    }
  },
  weekFlagTime: {
    type: Number,
    default: new Date().getTime()
  },
  pWeekdays: {
    type: Array,
    default() {
      return [[], [], [], [], [], [], []]
    }
  },
  weekOrMonthFlag: { // 周视图、月视图的标志
    type: String,
    default: 'week',// month
  },
  activities: {
    type: Array,
    default() {
      return []
    }
  }
})
const yearAndMonth = ref(props.time)
const weekTimeStamp = ref(props.weekFlagTime)
const emit = defineEmits(['update:time', 'update:weekFlagTime', 'update:pWeekdays', 'onActivityClick'])

// 将时间设置为今天
const setToday = () => {
  yearAndMonth.value.year = new Date().getFullYear();
  yearAndMonth.value.month = new Date().getMonth()
  emit('update:time', yearAndMonth.value)
}
// 上周或者上月
const leftArrow = () => {
  console.log(props.weekOrMonthFlag);
  props.weekOrMonthFlag === 'week' ? nextOrPreWeek(-1) : changeMonth(-1)

}
// 下周或者下月
const rightArrow = () => {
  props.weekOrMonthFlag === 'week' ? nextOrPreWeek(1) : changeMonth(1)

}


// 改变月份
const changeMonth = (value) => {
  if (value > 0) {
    if (yearAndMonth.value.month === 11) {
      yearAndMonth.value.year++
      yearAndMonth.value.month = 0
    } else {
      yearAndMonth.value.month++
    }
  } else if (value < 0) {
    if (yearAndMonth.value.month === 0) {
      yearAndMonth.value.year--
      yearAndMonth.value.month = 11
    } else {
      yearAndMonth.value.month--
    }
  }
  emit('update:time', yearAndMonth.value)
}

// 根据这周第一天获取这周的活动
const getWeekActivity = (weekStart) => {
  let filter = toRaw(props.activities)
  // 筛选出来的活动
  filter = filter.filter(item => {
    return item.beginTime.getTime() <= (weekStart.getTime() + 6 * 24 * 60 * 60 * 1000) && item.endTime.getTime() >= weekStart.getTime()
  }).sort((a, b) => {
    if (a.beginTime.getTime() < b.beginTime.getTime()) return -1 // 如果a的开始时间 大于 b的开始时间，那么a、b就不交换位置
    if (a.beginTime.getTime() > b.beginTime.getTime()) return 1
    if (a.endTime.getTime() < b.endTime.getTime()) return -1
    if (a.endTime.getTime() > b.endTime.getTime()) return 1
    return 0
  }).map(item => {
    item.beginTime.setHours(0)
    item.beginTime.setMinutes(0)
    item.beginTime.setSeconds(0)
    item.beginTime.setMilliseconds(0)
    item.endTime.setHours(0)
    item.endTime.setMinutes(0)
    item.endTime.setSeconds(0)
    item.endTime.setMilliseconds(0)
    return item
  })
  let topArr = [[], [], [], [], [], [], []] // 每个数组元素代表一天
  for (let i = 0; i < filter.length; i++) {
    // 给活动对象添加一个属性classes，保存class类
    const activity = filter[i]
    activity.classes = []
    activity.topVal = 0
    // 定义变量 = 活动的开始日 < 这周开始，为true代表这个活动是continuted（连续的）
    const continued = activity.beginTime.getTime() < weekStart.getTime()
    // 如果是continuted，该活动的offset为0；如果不是，offset为活动开始日 - 这周开始
    const startOffset = continued ? 0 : (activity.beginTime.getTime() - weekStart.getTime()) / (24 * 60 * 60 * 1000)

    // 如果是连续的，说明活动不是在这周第一次出现。添加连续或者活动开始的类。
    if (continued) {
      activity.classes.push("continuted")
    } else {
      activity.classes.push("begin-activity")
    }
    // 如果活动结束日距离这周开始大于6天，说明活动在这周内不会结束结束
    let toBeContinuted = (activity.endTime.getTime() - weekStart.getTime()) / (24 * 60 * 60 * 1000) > 6
    if (toBeContinuted) {
      activity.classes.push("toBeContinuted")
    } else {
      activity.classes.push("end-activity")
    }
    // span为活动在这周的持续天数
    let span = 0
    if (continued) {
      if (toBeContinuted) {
        span = 7
      } else {
        span = (activity.endTime.getTime() - weekStart.getTime()) / (24 * 60 * 60 * 1000) + 1
      }
    } else {
      if (toBeContinuted) {
        span = 7 - [7, 1, 2, 3, 4, 5, 6][activity.beginTime.getDay()] + 1
      } else {
        span = (activity.endTime.getTime() - activity.beginTime.getTime()) / (24 * 60 * 60 * 1000) + 1
      }
    }
    // 设置每个活动的top值
    for (let d = 0; d < 7; d++) {
      if (d === startOffset) {
        let n = 0
        while (topArr[d][n]) n++
        activity.topVal = n
        topArr[d][n] = true
      } else if (d < startOffset + span) {
        topArr[d][activity.topVal] = true
      }
    }
    // activity.classes.push(`activity-${activity.color}`)
    activity.classes.push(`offset${startOffset}`)
    activity.classes.push(`span${span}`)
    activity.classes.push(`activity-${activity.color}`)
    activity.classes.push(`top-${activity.topVal}`)
  }
  return filter
}
// 日历中显示的日期
const visibleCalendar = computed(() => {
  // 获取今天的年月日
  const today = new Date()
  today.setHours(0)
  today.setMinutes(0)
  today.setSeconds(0)
  today.setMilliseconds(0)
  // 定义最终返回的日历数组
  const calendarArr = [[], [], [], [], [], []]
  // 获取当前月份第一天
  const currentFirstDay = new Date(props.time.year, props.time.month, 1)
  // 获取第一天是周几
  const weekDay = currentFirstDay.getDay() // 0，1，2，3，4，5，6 ，周日代表0
  // 用当前月份第一天减去周几前面几天，就是看见的日历的第一天
  const startDay = currentFirstDay - ([7, 1, 2, 3, 4, 5, 6][weekDay] - 1) * 24 * 3600 * 1000
  // 我们统一用42天来显示当前显示日历
  for (let i = 0; i < 42; i++) {
    const date = new Date(startDay + i * 24 * 3600 * 1000)
    const floorIndex = Math.floor(i / 7)
    calendarArr[floorIndex].push({
      date, // 完整时间
      year: date.getFullYear(),// 年份
      month: date.getMonth(), // 月份
      day: ['周日', '周一', '周二', '周三', '周四', '周五', '周六'][date.getDay()], // 周几
      monthDay: date.getDate(), // 一个月中的第几天
      // 是否在当月
      thisMonth: date.getFullYear() === today.getFullYear() && date.getMonth() === today.getMonth(),
      // 是否在今天
      isToday: date.getFullYear() === today.getFullYear() && date.getMonth() === today.getMonth() && date.getDate() === today.getDate(),
      // 是否在今天之后
      afterToday: date.getTime() > today.getTime(),
      //  是否在今天之前
      beforeToday: date.getTime() < today.getTime(),
      // 是否是周六
      isSaturday: date.getDay() === 6, // 0代表周日, 6代表
      // 是否是周末
      isSunday: date.getDay() === 0
    })
  }
  return calendarArr
})
// weekdays
// 把42天分为六周。0，1，2，3，4，5
// 下周或者上周
const nextOrPreWeek = (value) => {
  weekTimeStamp.value += value * 7 * 24 * 3600 * 1000
  emit('update:pWeekdays', weekdays.value)
}

// 周视图下显示的日历，和visibleCalendar计算属性里的代码有部分一样
const weekdays = computed(() => {
  // 获取今天的年月日
  const today = new Date()
  const flagTime = new Date(weekTimeStamp.value)
  flagTime.setHours(0)
  flagTime.setMinutes(0)
  flagTime.setSeconds(0)
  flagTime.setMilliseconds(0)
  // 定义最终返回的周数组
  const weekArr = []
  // 获取今天是星期几
  const dayOfWeek = flagTime.getDay() // 0,1,2,3,4,5,6
  // 获取这周的第一天
  const startDay = flagTime.getTime() - ([7, 1, 2, 3, 4, 5, 6][dayOfWeek] - 1) * 24 * 3600 * 1000
  for (let i = 0; i < 7; i++) {
    const date = new Date(startDay + i * 24 * 3600 * 1000)
    weekArr.push({
      date, // 完整时间
      year: date.getFullYear(),// 年份
      month: date.getMonth(), // 月份
      day: ['周日', '周一', '周二', '周三', '周四', '周五', '周六'][date.getDay()], // 周几
      monthDay: date.getDate(), // 一个月中的第几天
      // 是否在当月
      thisMonth: date.getFullYear() === today.getFullYear() && date.getMonth() === today.getMonth(),
      // 是否在今天
      isToday: date.getFullYear() === today.getFullYear() && date.getMonth() === today.getMonth() && date.getDate() === today.getDate(),
      // 是否在今天之后
      afterToday: date.getTime() > today.getTime(),
      //  是否在今天之前
      beforeToday: date.getTime() < today.getTime(),
      // 是否是周六
      isSaturday: date.getDay() === 6, // 0代表周日, 6代表
      // 是否是周末
      isSunday: date.getDay() === 0
    })
  }
  return weekArr
})

// 暴露
defineExpose({ setToday, leftArrow, rightArrow, weekdays })
onMounted(() => {
  emit('update:pWeekdays', weekdays.value)
})

// 点击活动触发的事件
const onActivityClick = (activity) => {
  emit('onActivityClick', toRaw(activity))
}
</script>

<template>
  <!-- 周一 到 周日 -->
  <div class="week-day">
    <div style="display:flex;align-items:center" v-for="item in weekdays">
      <span v-if="weekOrMonthFlag === 'week'" :class="item.isToday ? 'item-today' : ''">{{ item.monthDay }}</span>
      <span v-if="(item.isSaturday || item.isSunday) && weekOrMonthFlag === 'week'"
        style="background-color:rgba(254, 44, 85, 0.1);color: rgba(254,44,85,.9);padding: 0 2px;">{{ item.day }}</span>
      <span v-else>{{ item.day }}</span>
    </div>
  </div>
  <!-- 日历 -->
  <div style="display:inline-block;" class="grid-container-wrapper">
    <!-- 周视图日历 -->
    <div v-if="weekOrMonthFlag === 'week'" class="gird-week">
      <div class="item" :style="`background-color: ${item.isToday ? '#F6F8FA' : ''};`"
        v-for="(item, index2) in weekdays"></div>
      <span class="activity" :class="activity.classes" :style="`top:${(activity.topVal * 35 + 45)}px;`"
        v-for="(activity, index3) in getWeekActivity(weekdays[0].date)">{{ activity.title }}</span>
    </div>
    <!-- 月视图日历 -->
    <div v-else>
      <div style="overflow:overlay" class="grid-month" v-for="(floor, index) in visibleCalendar">
        <div class="item" v-for="(item, index2) in floor">
          <div class>
            <span :class="[!item.thisMonth ? 'text-grey' : 'text-black', item.isToday ? 'item-today' : '']">{{
                item.monthDay
            }}</span>
            <span style="background-color:rgba(254, 44, 85, 0.1);color: rgba(254,44,85,.9);padding: 0 2px;"
              class="on-right text-weight-bold rounded" v-if="item.isSaturday">周六</span>
            <span style="background-color:rgba(254, 44, 85, 0.1);color: rgba(254,44,85,.9);padding: 0 2px;"
              class="on-right text-weight-bold rounded" v-if="item.isSunday">周日</span>
          </div>
        </div>
        <span @click="onActivityClick(activity)" class="activity" :class="activity.classes"
          :style="`top:${(activity.topVal * 35 + 45)}px;`"
          v-for="(activity, index3) in getWeekActivity(floor[0].date)">{{ activity.title }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
* {
  box-sizing: border-box;
}

.my-btn {
  border: 1px solid #ccc;
  border-radius: 2px;
  padding: 4px;
}

.text-grey {
  color: #9e9e9e;
}

.text-black {
  color: black;
  font-weight: 700;
}

.week-day {
  display: grid;
  grid-template-columns: repeat(7, 150px);
  grid-template-rows: repeat(1, 50px);
}

.item-today {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 35px;
  height: 35px;
  background-color: #221f20;
  color: #fff;
  padding: 10px;
  border-radius: 100px;
}

.gird-week {
  position: relative;
  display: inline-grid;
  grid-template-columns: repeat(7, 150px);
  grid-template-rows: repeat(1, calc(6 * 140px));
  border-left: 0.5px solid #ccc;
  border-top: 0.5px solid #ccc;

  .item {
    position: relative;
    box-sizing: border-box;
    border-right: 0.5px solid #ccc;
    padding: 8px;
  }
}

.grid-month {
  position: relative;
  display: grid;
  grid-template-columns: repeat(7, 150px);
  grid-template-rows: repeat(1, 148px);
  border-left: 0.5px solid #ccc;

  &:nth-child(1) {
    border-top: 0.5px solid #ccc;
  }

  .item {
    position: relative;
    box-sizing: border-box;
    padding: 8px;

    &::after {
      content: "";
      display: block;
      position: absolute;
      top: 0;
      right: 0;
      bottom: 0;
      left: 0;
      border-right: 0.5px solid #ccc;
      border-bottom: 0.5px solid #ccc;
    }
  }
}

.activity {
  position: absolute;
  left: 0;
  /* width: 100%; */
  height: 30px;
  padding: 5px 30px 0;
  font-weight: 600;
  z-index: 1;
  font-size: 0.8em;

  &.activity-red {
    background-color: rgba(236, 73, 73, 0.15);
    border: 1px solid rgba(236, 73, 73, 0.9);
    color: rgba(254, 44, 85, 0.9);
  }

  &.activity-blue {
    background: rgba(36, 83, 178, 0.15);
    border: 1px solid rgb(36, 83, 178);
    color: rgb(36, 83, 178);
  }

  &.activity-green {
    background: rgba(0, 230, 118, 0.15);
    border: 1px solid rgba(0, 230, 118, 1);
    color: rgba(0, 230, 118, 1);
  }

  &.begin-activity {
    border-top-left-radius: 10px;
    border-bottom-left-radius: 10px;
  }

  &.end-activity {
    border-top-right-radius: 10px;
    border-bottom-right-radius: 10px;
  }

  &.continuted {
    border-left-color: transparent;
  }

  &.toBeContinuted {
    border-right-color: transparent;
  }
}

/* 偏移 */
.activity.offset0 {
  left: 0;
}

.activity.offset1 {
  left: calc((100% / 7));
}

.activity.offset2 {
  left: calc((200% / 7));
}

.activity.offset3 {
  left: calc((300% / 7));
}

.activity.offset4 {
  left: calc((400% / 7));
}

.activity.offset5 {
  left: calc((500% / 7));
}

.activity.offset6 {
  left: calc((600% / 7));
}

/* 占据空间 */

.activity.span1 {
  width: calc((100% / 7));
}

.activity.span2 {
  width: calc((200% / 7));
}

.activity.span3 {
  width: calc((300% / 7));
}

.activity.span4 {
  width: calc((400% / 7));
}

.activity.span5 {
  width: calc((500% / 7));
}

.activity.span6 {
  width: calc((600% / 7));
}

.activity.span7 {
  width: calc((700% / 7));
}
</style>