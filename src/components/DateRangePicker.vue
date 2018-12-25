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
          <calendar-ranges @clickRange="clickRange" :ranges="ranges"></calendar-ranges>
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
        validator: prop => typeof prop === 'number' || prop === null
      },
      ranges: {
        type: Object,
        default() {
          return {
            'Yesterday': [moment().add(-1, 'day').startOf('day'), moment().add(-1, 'day').endOf('day'), 1],
            'Today': [moment().startOf('day'), moment().endOf('day'), 'today', 2],
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
      }
    }
  }

</script>

<style lang="scss">















  .daterangepicker {
    position: absolute;
    color: inherit;
    background-color: #fff;
    border-radius: 4px;
    border: 1px solid #ddd;
    width: 278px;
    max-width: none;
    padding: 0;
    margin-top: 7px;
    top: 100px;
    left: 20px;
    z-index: 3001;
    display: none;
    font-size: 15px;
    line-height: 1em;
  }

  .daterangepicker:before, .daterangepicker:after {
    position: absolute;
    display: inline-block;
    border-bottom-color: rgba(0, 0, 0, 0.2);
    content: '';
  }

  .daterangepicker:before {
    top: -7px;
    border-right: 7px solid transparent;
    border-left: 7px solid transparent;
    border-bottom: 7px solid #ccc;
  }

  .daterangepicker:after {
    top: -6px;
    border-right: 6px solid transparent;
    border-bottom: 6px solid #fff;
    border-left: 6px solid transparent;
  }

  .daterangepicker.opensleft:before {
    right: 9px;
  }

  .daterangepicker.opensleft:after {
    right: 10px;
  }

  .daterangepicker.openscenter:before {
    left: 0;
    right: 0;
    width: 0;
    margin-left: auto;
    margin-right: auto;
  }

  .daterangepicker.openscenter:after {
    left: 0;
    right: 0;
    width: 0;
    margin-left: auto;
    margin-right: auto;
  }

  .daterangepicker.opensright:before {
    left: 9px;
  }

  .daterangepicker.opensright:after {
    left: 10px;
  }

  .daterangepicker.drop-up {
    margin-top: -7px;
  }

  .daterangepicker.drop-up:before {
    top: initial;
    bottom: -7px;
    border-bottom: initial;
    border-top: 7px solid #ccc;
  }

  .daterangepicker.drop-up:after {
    top: initial;
    bottom: -6px;
    border-bottom: initial;
    border-top: 6px solid #fff;
  }

  .daterangepicker.single .daterangepicker .ranges, .daterangepicker.single .drp-calendar {
    float: none;
  }

  .daterangepicker.single .drp-selected {
    display: none;
  }

  .daterangepicker.show-calendar .drp-calendar {
    display: block;
  }

  .daterangepicker.show-calendar .drp-buttons {
    display: block;
  }

  .daterangepicker.auto-apply .drp-buttons {
    display: none;
  }

  .daterangepicker .drp-calendar {
    display: none;
    max-width: 270px;
  }

  .daterangepicker .drp-calendar.left {
    padding: 8px 0 8px 8px;
  }

  .daterangepicker .drp-calendar.right {
    padding: 8px;
  }

  .daterangepicker .drp-calendar.single .calendar-table {
    border: none;
  }

  .daterangepicker .calendar-table .next span, .daterangepicker .calendar-table .prev span {
    color: #fff;
    border: solid black;
    border-width: 0 2px 2px 0;
    border-radius: 0;
    display: inline-block;
    padding: 3px;
  }

  .daterangepicker .calendar-table .next span {
    transform: rotate(-45deg);
    -webkit-transform: rotate(-45deg);
  }

  .daterangepicker .calendar-table .prev span {
    transform: rotate(135deg);
    -webkit-transform: rotate(135deg);
  }

  .daterangepicker .calendar-table th, .daterangepicker .calendar-table td {
    white-space: nowrap;
    text-align: center;
    vertical-align: middle;
    min-width: 32px;
    width: 32px;
    height: 24px;
    line-height: 24px;
    font-size: 12px;
    border-radius: 4px;
    border: 1px solid transparent;
    white-space: nowrap;
    cursor: pointer;
  }

  .daterangepicker .calendar-table {
    border: 1px solid #fff;
    border-radius: 4px;
    background-color: #fff;
  }

  .daterangepicker .calendar-table table {
    width: 100%;
    margin: 0;
    border-spacing: 0;
    border-collapse: collapse;
  }

  .daterangepicker td.available:hover, .daterangepicker th.available:hover {
    background-color: #eee;
    border-color: transparent;
    color: inherit;
  }

  .daterangepicker td.week, .daterangepicker th.week {
    font-size: 80%;
    color: #ccc;
  }

  .daterangepicker td.off, .daterangepicker td.off.in-range, .daterangepicker td.off.start-date, .daterangepicker td.off.end-date {
    background-color: #fff;
    border-color: transparent;
    color: #999;
  }

  .daterangepicker td.in-range {
    background-color: #ebf4f8;
    border-color: transparent;
    color: #000;
    border-radius: 0;
  }

  .daterangepicker td.start-date {
    border-radius: 4px 0 0 4px;
  }

  .daterangepicker td.end-date {
    border-radius: 0 4px 4px 0;
  }

  .daterangepicker td.start-date.end-date {
    border-radius: 4px;
  }

  .daterangepicker td.active, .daterangepicker td.active:hover {
    background-color: #357ebd;
    border-color: transparent;
    color: #fff;
  }

  .daterangepicker th.month {
    width: auto;
  }

  .daterangepicker td.disabled, .daterangepicker option.disabled {
    color: #999;
    cursor: not-allowed;
    text-decoration: line-through;
  }

  .daterangepicker select.monthselect, .daterangepicker select.yearselect {
    font-size: 12px;
    padding: 1px;
    height: auto;
    margin: 0;
    cursor: default;
  }

  .daterangepicker select.monthselect {
    margin-right: 2%;
    width: 56%;
  }

  .daterangepicker select.yearselect {
    width: 40%;
  }

  .daterangepicker select.hourselect, .daterangepicker select.minuteselect, .daterangepicker select.secondselect, .daterangepicker select.ampmselect {
    width: 50px;
    margin: 0 auto;
    background: #eee;
    border: 1px solid #eee;
    padding: 2px;
    outline: 0;
    font-size: 12px;
  }

  .daterangepicker .calendar-time {
    text-align: center;
    margin: 4px auto 0 auto;
    line-height: 30px;
    position: relative;
  }

  .daterangepicker .calendar-time select.disabled {
    color: #ccc;
    cursor: not-allowed;
  }

  .daterangepicker .drp-buttons {
    clear: both;
    text-align: right;
    padding: 8px;
    border-top: 1px solid #ddd;
    display: none;
    line-height: 12px;
    vertical-align: middle;
  }

  .daterangepicker .drp-selected {
    display: inline-block;
    font-size: 12px;
    padding-right: 8px;
  }

  .daterangepicker .drp-buttons .btn {
    margin-left: 8px;
    font-size: 12px;
    font-weight: bold;
    padding: 4px 8px;
  }

  .daterangepicker.show-ranges .drp-calendar.left {
    border-left: 1px solid #ddd;
  }

  .daterangepicker .ranges {
    float: none;
    text-align: left;
    margin: 0;
  }

  .daterangepicker.show-calendar .ranges {
    margin-top: 8px;
  }

  .daterangepicker .ranges ul {
    list-style: none;
    margin: 0 auto;
    padding: 0;
    width: 100%;
  }

  .daterangepicker .ranges li {
    font-size: 12px;
    padding: 8px 12px;
    cursor: pointer;
  }

  .daterangepicker .ranges li:hover {
    background-color: #eee;
  }

  .daterangepicker .ranges li.active {
    background-color: #08c;
    color: #fff;
  }

  /*  Larger Screen Styling */
  @media (min-width: 564px) {
    .daterangepicker {
      width: auto; }
    .daterangepicker .ranges ul {
      width: 140px; }
    .daterangepicker.single .ranges ul {
      width: 100%; }
    .daterangepicker.single .drp-calendar.left {
      clear: none; }
    .daterangepicker.single.ltr .ranges, .daterangepicker.single.ltr .drp-calendar {
      float: left; }
    .daterangepicker.single.rtl .ranges, .daterangepicker.single.rtl .drp-calendar {
      float: right; }
    .daterangepicker.ltr {
      direction: ltr;
      text-align: left; }
    .daterangepicker.ltr .drp-calendar.left {
      clear: left;
      margin-right: 0; }
    .daterangepicker.ltr .drp-calendar.left .calendar-table {
      border-right: none;
      border-top-right-radius: 0;
      border-bottom-right-radius: 0; }
    .daterangepicker.ltr .drp-calendar.right {
      margin-left: 0; }
    .daterangepicker.ltr .drp-calendar.right .calendar-table {
      border-left: none;
      border-top-left-radius: 0;
      border-bottom-left-radius: 0; }
    .daterangepicker.ltr .drp-calendar.left .calendar-table {
      padding-right: 8px; }
    .daterangepicker.ltr .ranges, .daterangepicker.ltr .drp-calendar {
      float: left; }
    .daterangepicker.rtl {
      direction: rtl;
      text-align: right; }
    .daterangepicker.rtl .drp-calendar.left {
      clear: right;
      margin-left: 0; }
    .daterangepicker.rtl .drp-calendar.left .calendar-table {
      border-left: none;
      border-top-left-radius: 0;
      border-bottom-left-radius: 0; }
    .daterangepicker.rtl .drp-calendar.right {
      margin-right: 0; }
    .daterangepicker.rtl .drp-calendar.right .calendar-table {
      border-right: none;
      border-top-right-radius: 0;
      border-bottom-right-radius: 0; }
    .daterangepicker.rtl .drp-calendar.left .calendar-table {
      padding-left: 12px; }
    .daterangepicker.rtl .ranges, .daterangepicker.rtl .drp-calendar {
      text-align: right;
      float: right; } }
  @media (min-width: 730px) {
    .daterangepicker .ranges {
      width: auto; }
    .daterangepicker.ltr .ranges {
      float: left; }
    .daterangepicker.rtl .ranges {
      float: right; }
    .daterangepicker .drp-calendar.left {
      clear: none !important; } }












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
