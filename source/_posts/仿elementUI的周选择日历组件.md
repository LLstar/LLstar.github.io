---
title: 仿elementUI的周选择日历组件
date: 2019-11-06 15:19:48
tags:
- vue
- element-ui
categories:
- Web前端
copyright: true
---

日历的控件数不胜数，大多数日历控件都是用来选择日期，用来找出具体的某一天，但是我们的需求是需要点击本周的任意一天，选择本周的时间，找了很多组件，并没有发现合适的组件，于是决定再造一个轮子，自己仿照elementUI中的日期选择选择周的日期组件，来封装一个选择周的日历组件。

UI图样式如下：UI样式是仿照elementUI中的日期选择器（element的样式比较好看）

<!-- more -->

![](https://i.loli.net/2019/11/07/6XfbFndIAE4gzUi.png)

### 实现分析

​		观察到日历的排列为一个7X6的布局，保证日历的显示是定宽和定高的，不会随着月份的改变而更改，而且7X6能保证该月的日期都能正常显示。如何将当月的日期在日期中绘制出来？观察发现，日历的第一周是当月的1号所在的周，结束日期找到当月的最后一天所在周的最后一天，从开始周的第一天到最后周的最后一天中间如果相隔35天，则再结束日期再向后顺延一周，否则就是当前选择的结束日期。

    // 开始日期 = 当月1号所在周的开始日期
    // 结束日期 = 结束日期 - 开始日期 === 41 ？ 当月最后日期所在周的最后一天 ： 当月最后日期所在周的最后一天 + 7天

#### 基本结构

    <template>
      <div class="weekDay">
        <div class="date-header">
          <button
            type="button"
            aria-label="前一年"
            class="el-picker-panel__icon-btn el-date-picker__prev-btn el-icon-d-arrow-left"
            @click="currentYear -= 1"
          ></button>
          <button
            type="button"
            aria-label="上个月"
            class="el-picker-panel__icon-btn el-date-picker__prev-btn el-icon-arrow-left"
            @click="currentMonth -= 1"
          ></button>
          <span role="button" class="el-date-picker__header-label">{{ currentYear }}年</span>
          <span role="button" class="el-date-picker__header-label">{{ currentMonth }}月</span>
          <button
            type="button"
            aria-label="后一年"
            class="el-picker-panel__icon-btn el-date-picker__next-btn el-icon-d-arrow-right"
            @click="currentYear += 1"
          ></button>
          <button
            type="button"
            aria-label="下个月"
            class="el-picker-panel__icon-btn el-date-picker__next-btn el-icon-arrow-right"
            @click="currentMonth += 1"
          ></button>
        </div>
        <table cellspacing="0" cellpadding="0" class="date-table">
          <tbody>
            <tr>
              <th v-for="(week, key) in WEEKS" :key="key">{{ week }}</th>
            </tr>
            <tr
              v-for="(row, key) in rows"
              :key="key"
              :class="activeClass(row)"
              @click="chooseDate(row, key)"
            >
              <td v-for="(cell, key) in row" :key="key">
                <div :class="disabledDayStyle(cell)" style="padding: 3px 0;box-sizing: border-box;">
                  <span class="cell-table">{{ new Date(cell).getDate() }}</span>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </template>

#### 绘制日历

- 计算一个月中的开始日期和结束日期

    ```js
    created() {
        this.firstDate = new Date(dayjs().startOf('month')) //当月的第一天
        this.lastDate = new Date(dayjs().endOf('month')) // 当月的最后一天
    },
    // 某天所在周的周一
     weekFirstDay(date) {
        if (new Date(date).getDay() !== 0) {
            const weekfirst = dayjs(date)
            .startOf('week')
            .add(1, 'day')
            return weekfirst
        }
        const weekfirst = dayjs(date)
        .subtract(1, 'day')
        .startOf('week')
        .add(1, 'day')
        return weekfirst
     },
     // 某天所在周的周日
     weekLastDay(date) {
         const weeklast = dayjs(date)
         .endOf('week')js
         .add(1, 'day')
         return weeklast
     }
    ```

由于要处理很多的时间，用js中的date处理起来比较麻烦，因此直接使用day.js来对日期进行统一处理。

由于产品需要，一周以周一开始，以周日结束。

- 绘制一月中的日期

    ```js
    // 上面已经可以计算出绘制的日期的开始时间和结束时间，那么需要进行遍历求出当月包含当月所有日期的数组。
    // 获取天数的数组
    getDays() {
        this.days = []
        let startDate = this.weekFirstDay(this.firstDate) // 开始日期
        let endDate = this.weekLastDay(this.lastDate) // 结束日期
        const diffDay = endDate.diff(startDate, 'day') // 差值
        if (diffDay !== 41) {
            // 如果差值不为41天，则向后加7天
            endDate = dayjs(endDate).add(7, 'day')
        }
        // 遍历后组成一个包含所有日期的数组
        while (startDate <= endDate) {
            this.days.push(dayjs(startDate).toDate())
            startDate = dayjs(startDate)
                .add(1, 'day')
                .toDate()
        }
    }，
    computed: {
        // 利用计算属性，对该数组进行遍历，如果到7，就自动变为下一行，将该一维数组变为一个二维数组
        rows() {
            let childArr = []
            const parentArr = []
            this.days.forEach(day => {
                childArr.push(day)
                if (childArr.length === 7) {
                    parentArr.push(childArr)
                    childArr = []
                }
            })
            return parentArr
        }
    }
    ```

在此过程中需要将当前天的日期标记出来，同时需要对所选周的周一及周日进行高亮显示。

- 切换月份和年份的实现

上面已经基本上把一个日历绘制出来了，切换月份和年份就比较简单一点。根据改变的年份和月份重新计算所选年月的日期数组，进行遍历即可。

    // 日期改变
    changeDate() {
        const now = new Date(`${this.currentYear}-${this.currentMonth}`)
        this.firstDate = new Date(dayjs(now).startOf('month'))
        this.lastDate = new Date(dayjs(now).endOf('month'))
        this.getDays()
    }，
    watch: {
      currentYear: {
         handler() {
           this.changeDate()
         },
         deep: true
      },
      currentMonth: {
        handler(newMon) {
          if (newMon === 0) {
            this.currentMonth = 12
            this.currentYear -= 1
          } else if (newMon === 13) {
            this.currentMonth = 1
            this.currentYear += 1
          }
          this.changeDate()
        },
        deep: true
       }
     }

以上就是绘制这个日历周选择控件的基本思路，只要可以找到所选择月的开始日期和结束日期，找出需要绘制的日期的数据，对数据进行遍历绘制即可。剩下的就是一些UI的样式问题。仿照一个好看的日历组件，修改一下样式即可。
