<template>
  <div class="schedule-calendar">
    <div class="title">
      <span>日程</span>
      <div class="rt"
           @click="addSchedule">
        <span class="el-icon-plus"></span>
        <span>创建</span>
      </div>
    </div>
    <div>
      <Calendar :markDateMore="calendarArr"
                @changeMonth="changeMonth"
                @choseDay="clickDay">
      </Calendar>
      <div v-loading="loading">
        <div class="list"
             v-for="(item, index) in scheduleList"
             @click="rowFun(item)"
             :key="index"
             v-if="index < 1">
          <p class="list-title">{{item.title}}</p>
          <div>
            <span class="time">{{item.start_time | moment("YYYY-MM-DD")}} - {{item.end_time | moment("YYYY-MM-DD")}}</span>
            <template v-if="item.ownList.length != 0">
              <span v-for="(k, j) in item.ownList"
                    :key="j">{{item.ownList.length - 1 == j ? k.realname : k.realname + '、' }}</span>
            </template>
          </div>
        </div>
        <p v-if="scheduleList.length >= 1"
           @click="seeMore"
           class="see-more">查看更多</p>
      </div>
    </div>
    <!-- 新建日程 -->
    <create-schedule v-if="showDialog"
                     :text="newText"
                     :formData="formData"
                     @onSubmit="onSubmit"
                     @closeDialog="closeDialog">
    </create-schedule>
  </div>
</template>

<script>
import Calendar from 'vue-calendar-component'
import createSchedule from '../schedule/components/createSchedule'
import { scheduleListAPI } from '@/api/oamanagement/workbench'

export default {
  components: {
    Calendar,
    createSchedule
  },
  data() {
    return {
      // 日历事件
      scheduleList: [],
      // 新建日程
      formData: {},
      showDialog: false,
      newText: '',
      // 详情
      dialogVisible: false,
      loading: false
    }
  },
  props: {
    calendarArr: {
      type: Array,
      default: () => {
        return []
      }
    }
  },
  mounted() {
    this.clickDay(new Date())
  },
  methods: {
    // 点击哪一天
    clickDay(date) {
      this.loading = true
      scheduleListAPI({ start_time: Date.parse(date) / 1000 })
        .then(res => {
          this.scheduleList = res.data
          for (let item of document.getElementsByClassName('wh_item_date')) {
            item.classList.remove('wh_isToday')
          }
          this.loading = false
        })
        .catch(error => {
          this.loading = false
        })
    },
    // 查看更多
    seeMore() {
      this.$router.push('schedule')
    },
    // 创建日程
    addSchedule() {
      this.showDialog = true
      this.newText = '创建日程'
    },
    // 关闭新建日程弹框
    closeDialog() {
      this.showDialog = false
    },
    onSubmit() {
      this.closeDialog()
    },
    // 查看详情
    rowFun(val) {
      this.dialogVisible = true
    },
    // 切换月份
    changeMonth(val) {
      this.scheduleList = []
      // 初始化时间
      let now = new Date(val)
      // 获取当月第一天
      var nowYear = now.getYear() //当前年
      nowYear += nowYear < 2000 ? 1900 : 0
      var nowMonth = now.getMonth() //当前月
      var monthStartDate = new Date(nowYear, nowMonth, 1)
      // 获取当月最后一天
      var monthEndDate = new Date(nowYear, nowMonth + 1, 1)
      var days = (monthEndDate - monthStartDate) / (1000 * 60 * 60 * 24)
      var lastDay = new Date(nowYear, nowMonth, days)
      this.$emit(
        'changeMonth',
        monthStartDate.getTime() / 1000,
        lastDay.getTime() / 1000
      )
    }
  }
}
</script>

<style scoped lang="scss">
.schedule-calendar {
  padding-bottom: 20px;
  .title {
    height: 44px;
    line-height: 44px;
    border-bottom: 1px solid #e6e6e6;
    .rt {
      color: #3e84e9;
      margin-right: 0;
      cursor: pointer;
      .el-icon-plus {
        font-weight: 700;
      }
    }
  }
  .list {
    color: #999;
    padding: 10px 11%;
    font-size: 13px;
    cursor: pointer;
    .list-title {
      margin-bottom: 5px;
    }
    .time {
      margin-right: 10px;
      font-size: 12px;
    }
  }
  .see-more {
    color: #999;
    text-align: center;
    cursor: pointer;
  }
}
.schedule-calendar /deep/ .wh_container {
  max-width: max-content;
  .wh_content_all {
    background: #fff;
    .wh_top_changge {
      text-align: center;
      display: block;
      margin: 15px 0 10px;
      li {
        color: #3c8be3;
        display: inline-block;
        height: 13px;
        .wh_jiantou1,
        .wh_jiantou2 {
          border-color: #3c8be3;
          width: 8px;
          height: 8px;
          margin-top: 4px;
        }
      }
      .wh_content_li {
        margin: 0 30px;
        font-weight: 700;
        font-size: 15px;
      }
    }
    .wh_content {
      justify-content: center;
      .wh_content_item {
        color: #333;
        font-size: 14px;
        margin-top: 3px;
        height: 30px;
        position: relative;
        .wh_top_tag {
          color: #bfbfbf;
        }
        .wh_item_date {
          background: transparent;
          width: 30px;
          height: 30px;
        }
        .wh_item_date:hover {
          background: transparent;
          color: #3c8be3;
        }
        .wh_isToday,
        .wh_isToday:hover {
          background: #3487e2;
          border-radius: 30px;
          color: #fff;
        }
        .wh_chose_day,
        .wh_chose_day:hover {
          background: #3487e2;
          border-radius: 40px;
          color: #fff;
        }
        .mark1:after {
          content: ' ';
          background-color: #3c8be3;
          width: 5px;
          height: 5px;
          position: absolute;
          bottom: 0;
          border-radius: 50%;
        }
      }
    }
  }
}
</style>
