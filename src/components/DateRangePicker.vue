<template>
  <div style="position: relative; display: inline-block; width: 100%">
    <div class="form-control" @click="togglePicker">
      <slot
        name="input"
        :startDate="start"
        :endDate="end"
        :ranges="ranges"
      >
        <i class="glyphicon glyphicon-calendar fa fa-calendar"></i>&nbsp;
        <span>{{startText}} - {{endText}}</span>
        <b class="caret"></b>
      </slot>
    </div>
    <transition name="slide-fade" mode="out-in">
      <div
        class="daterangepicker dropdown-menu ltr show-ranges"
        :class="pickerStyles()"
        v-if="open"
        v-on-clickaway="clickAway">
        <div class="calendars">
          <calendar-ranges @clickRange="clickRange" :ranges="ranges" :preset="preset"></calendar-ranges>
          <div class="drp-calendar left">
            <div class="daterangepicker_input hidden-xs" v-if="false">
              <input class="input-mini form-control" type="text" name="daterangepicker_start"
                     :value="startText"/>
              <i class="fa fa-calendar glyphicon glyphicon-calendar"></i>
            </div>
            <div class="calendar-table">
              <calendar :monthDate="monthDate"
                        :locale="locale"
                        :start="start" :end="end"
                        :minDate="min" :maxDate="max"
                        @nextMonth="nextMonth" @prevMonth="prevMonth"
                        @dateClick="dateClick" @hoverDate="hoverDate"
              ></calendar>
            </div>
          </div>

          <div class="drp-calendar right hidden-xs">
            <div class="daterangepicker_input" v-if="false">
              <input class="input-mini form-control" type="text" name="daterangepicker_end"
                     :value="endText"/>
              <i class="fa fa-calendar glyphicon glyphicon-calendar"></i>
            </div>
            <div class="calendar-table">
              <calendar :monthDate="nextMonthDate"
                        :locale="locale"
                        :start="start" :end="end"
                        :minDate="min" :maxDate="max"
                        @nextMonth="nextMonth" @prevMonth="prevMonth"
                        @dateClick="dateClick" @hoverDate="hoverDate"
              ></calendar>
            </div>
          </div>
        </div>

        <div class="drp-buttons">
          <button
            class="applyBtn btn btn-sm btn-success"
            :disabled="in_selection"
            type="button"
            @click="clickedApply"
          >{{locale.applyLabel}}
          </button>
          <button
            class="cancelBtn btn btn-sm btn-default"
            type="button"
            @click="open=false"
          >{{locale.cancelLabel}}
          </button>
        </div>

      </div>
    </transition>
  </div>
</template>

<script>
  import moment from 'moment'
  import Calendar from './Calendar.vue'
  import CalendarRanges from './CalendarRanges'
  import {nextMonth, prevMonth} from './util'
  import {mixin as clickaway} from 'vue-clickaway'

  export default {
    components: {Calendar, CalendarRanges},
    mixins: [clickaway],
    props: {
      minDate: [String, Object],
      maxDate: [String, Object],
      localeData: {
        type: Object,
        default() {
          return {}
        },
      },
      startDate: {
        default() {
          return new Date()
        }
      },
      endDate: {
        default() {
          return new Date()
        }
      },
      presetId: {
        default(){
          return null;
        }
      },
      ranges: {
        type: Object,
        default() {
          return {
            'Yesterday': [moment().add(-1, 'day').startOf('day'), moment().add(-1, 'day').endOf('day'), 1],
            'Today': [moment().startOf('day'), moment().endOf('day'), 2],
            'Tomorrow': [moment().add(1, 'day').startOf('day'), moment().add(1, 'day').endOf('day'), 3],

            'Last week': [moment().add(-1, 'week').startOf('week'), moment().add(-1, 'week').endOf('week'), 4],
            'This week': [moment().startOf('week'), moment().endOf('week'), 5],
            'Next week': [moment().add(1, 'week').startOf('week'), moment().add(1, 'week').endOf('week'), 6],

            'Last month': [moment().add(-1, 'month').startOf('month'), moment().add(-1, 'month').endOf('month'), 7],
            'This month': [moment().startOf('month'), moment().endOf('month'), 8],
            'Next month': [moment().add(1, 'month').startOf('month'), moment().add(1, 'month').endOf('month'), 9],

            'Last year': [moment().add(-1, 'year').startOf('year'), moment().add(-1, 'year').endOf('year'), 10],
            'This year': [moment().startOf('year'), moment().endOf('year'), 11],
            'Next year': [moment().add(1, 'year').startOf('year'), moment().add(1, 'year').endOf('year'), 12],
          }
        }
      },
      opens: {
        type: String,
        default: 'center'
      }
    },
    data() {
      let default_locale = {
        direction: 'ltr',
        format: moment.localeData().longDateFormat('L'),
        separator: ' - ',
        applyLabel: 'Apply',
        cancelLabel: 'Cancel',
        weekLabel: 'W',
        customRangeLabel: 'Custom Range',
        daysOfWeek: moment.weekdaysMin(),
        monthNames: moment.monthsShort(),
        firstDay: moment.localeData().firstDayOfWeek()
      }

      // let data = { locale: _locale }
      let data = {locale: {...default_locale, ...this.localeData}}

      data.monthDate = new Date(this.startDate)
      data.start = new Date(this.startDate)
      data.end = new Date(this.endDate)
      data.in_selection = false
      data.open = false
      data.preset = this.presetId

      // update day names order to firstDay
      if (data.locale.firstDay !== 0) {
        let iterator = data.locale.firstDay
        while (iterator > 0) {
          data.locale.daysOfWeek.push(data.locale.daysOfWeek.shift())
          iterator--
        }
      }
      return data
    },
    methods: {
      nextMonth() {
        this.monthDate = nextMonth(this.monthDate)
      },
      prevMonth() {
        this.monthDate = prevMonth(this.monthDate)
      },
      dateClick(value) {
        this.preset = null;
        if (this.in_selection) {
          this.in_selection = false;
          this.end = moment(value).endOf('day').toDate();
          if (this.end < this.start) {
            this.in_selection = true;
            this.start = moment(value).startOf('day').toDate();
          }
        } else {
          this.in_selection = true;
          this.start = moment(value).startOf('day').toDate();
          this.end = moment(value).endOf('day').toDate();
        }
      },
      hoverDate(value) {
        let dt = new Date(value)
        if (this.in_selection && dt > this.start)
          this.end = dt
      },
      togglePicker() {
        this.open = !this.open
      },
      pickerStyles() {
        return {
          'show-calendar': this.open,
          opensright: this.opens === 'right',
          opensleft: this.opens === 'left',
          openscenter: this.opens === 'center'
        }
      },
      clickedApply() {
        this.open = false
        this.$emit('update', {startDate: this.start, endDate: this.end, presetId: this.preset})
      },
      clickAway() {
        if (this.open) {
          this.open = false
        }
      },
      clickRange(value) {
        this.start = new Date(value[0])
        this.end = new Date(value[1])
        this.monthDate = new Date(value[0])
        this.preset = value[2]
        this.clickedApply()
      }
    },
    computed: {
      nextMonthDate() {
        return nextMonth(this.monthDate)
      },
      startText() {
        // return this.start.toLocaleDateString()
        return moment(this.start).format(this.locale.format)
      },
      endText() {
        // return new Date(this.end).toLocaleDateString()
        return moment(new Date(this.end)).format(this.locale.format)
      },
      min() {
        return this.minDate ? new Date(this.minDate) : null
      },
      max() {
        return this.maxDate ? new Date(this.maxDate) : null
      }
    },
    watch: {
      startDate(value) {
        this.start = new Date(value)
      },
      endDate(value) {
        this.end = new Date(value)
      },
      presetId(value) {
        this.preset = value
      },
    }
  }

</script>

<style lang="scss">
  @import '../assets/daterangepicker.css';
</style>

<style lang="scss" scoped>
  .reportrange-text {
    background: #fff;
    cursor: pointer;
    padding: 5px 10px;
    border: 1px solid #ccc;
    width: 100%;
  }

  .daterangepicker {
    flex-direction: column;
    display: flex;
    width: auto;

    &.show-calendar {
      display: block;
    }
  }

  .calendars {
    display: flex;
  }

  div.daterangepicker.opensleft {
    top: 35px;
    right: 10px;
    left: auto;
  }

  div.daterangepicker.openscenter {
    top: 35px;
    right: auto;
    left: 50%;
    transform: translate(-50%, 0);
  }

  div.daterangepicker.opensright {
    top: 35px;
    left: 10px;
    right: auto;
  }

  /* Enter and leave animations can use different */
  /* durations and timing functions.              */
  .slide-fade-enter-active {
    transition: all .2s ease;
  }

  .slide-fade-leave-active {
    transition: all .1s cubic-bezier(1.0, 0.5, 0.8, 1.0);
  }

  .slide-fade-enter, .slide-fade-leave-to
    /* .slide-fade-leave-active for <2.1.8 */
  {
    transform: translateX(10px);
    opacity: 0;
  }

</style>
