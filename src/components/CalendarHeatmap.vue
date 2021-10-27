<template lang="pug">
  svg.vch__wrapper(:viewBox="viewbox" ref="svg_element" :id="svgID" )
    text.temp_label(
      style="display:none"
    ) {{ "abc" }}
    g.vch__months__labels__wrapper(:transform="monthsLabelWrapperTransform[position]")
      text.vch__month__label(
        v-for="(month, index) in heatmap.firstFullWeekOfMonths",
        :x="getMonthLabelPostion(month).x",
        :y="getMonthLabelPostion(month).y"
      ) {{ lo.months[month.value] }}

    g.vch__days__labels__wrapper(:transform="daysLabelWrapperTransform[position]")
      text.vch__day__label(
        :x="vertical ? SQUARE_SIZE * 1 : 0",
        :y="vertical ? SQUARE_SIZE - SQUARE_BORDER_SIZE : 20"
      ) {{ lo.days[1] }}
      text.vch__day__label(
        :x="vertical ? SQUARE_SIZE * 3 : 0",
        :y="vertical ? SQUARE_SIZE - SQUARE_BORDER_SIZE : 44"
      ) {{ lo.days[3] }}
      text.vch__day__label(
        :x="vertical ? SQUARE_SIZE * 5 : 0",
        :y="vertical ? SQUARE_SIZE - SQUARE_BORDER_SIZE : 69"
      ) {{ lo.days[5] }}

    g.vch__legend__wrapper(:transform="legendWrapperTransform[position]" :key="legendKey")
      g(
        v-for="(color, index) in rangeColor",
        :key="index",
      )
        rect(
          :style="{ fill: color }",
          :width="SQUARE_SIZE - SQUARE_BORDER_SIZE",
          :height="SQUARE_SIZE - SQUARE_BORDER_SIZE",
          :x="vertical ? SQUARE_SIZE * 1.75 : getLegendRectVerticalXInPosition(index)",
          :y="vertical ? (SQUARE_SIZE * (index + 1)) + ((index == 0 ? 0 : 15) * index) : 5"
          v-tooltip="heatmap.getColorValue(index)",
        )
        text(
          v-if="isMobile"
          :x="vertical ? SQUARE_SIZE  : (SQUARE_SIZE - SQUARE_BORDER_SIZE) + 1 + getLegendRectVerticalXInPosition(index)",
          :y="vertical ? SQUARE_SIZE * (index + 1) + (SQUARE_SIZE * 1.75) + ((index == 0 ? 0 : 15) * index): SQUARE_SIZE + 1"
          :id="'svg_text_' + index"
          textLength="24"
          lengthAdjust="spacingAndGlyphs"
        ) {{ heatmap.getColorValue(index) }}
        text(
          v-else
          :x="vertical ? SQUARE_SIZE  : (SQUARE_SIZE - SQUARE_BORDER_SIZE) + 1 + getLegendRectVerticalXInPosition(index)",
          :y="vertical ? SQUARE_SIZE * (index + 1) + (SQUARE_SIZE * 1.75) + ((index == 0 ? 0 : 15) * index): SQUARE_SIZE + 1"
          :id="'svg_text_' + index"
        ) {{ heatmap.getColorValue(index) }}
    g.vch__year__wrapper(:transform="yearWrapperTransform")
      g.vch__month__wrapper(
        v-for="(week, weekIndex) in heatmap.calendar",
        :key="weekIndex",
        :transform="getWeekPosition(weekIndex)"
      )
        rect.vch__day__square(
          v-for="(day, dayIndex) in week",
          v-if="day.date < now"
          :key="dayIndex",
          :transform="getDayPosition(dayIndex)"
          :width="SQUARE_SIZE - SQUARE_BORDER_SIZE",
          :height="SQUARE_SIZE - SQUARE_BORDER_SIZE",
          :style="{ fill: rangeColor[day.colorIndex] }",
          v-tooltip="tooltipOptions(day)",
          @click="$emit('day-click', {day,content: returnTooltipContent(day)})"
        )
</template>

<script>
import { VTooltip } from 'v-tooltip'
import Heatmap from './Heatmap'
import { DAYS_IN_WEEK, DEFAULT_LOCALE, DEFAULT_RANGE_COLOR, DEFAULT_TOOLTIP_UNIT, SQUARE_SIZE } from './consts.js'

VTooltip.enabled = window.innerWidth > 768

export default {
  directives: {
    tooltip: VTooltip
  },

  props: {
    endDate: {
      required: true
    },
    max: {
      type: Number
    },
    rangeColor: {
      type: Array,
      default: () => DEFAULT_RANGE_COLOR
    },
    values: {
      required: true,
      type: Array
    },
    locale: {
      type: Object
    },
    tooltip: {
      type: Boolean,
      default: true
    },
    tooltipUnit: {
      type: String,
      default: DEFAULT_TOOLTIP_UNIT
    },
    // vertical: {
    //   type: Boolean,
    //   default: false
    // },
    noDataText: {
      type: String,
      default: null
    }
  },
  watch: {
    values: function(newVal, oldVal) { // watch it
      this.legendKey = new Date().getTime()
    }
  },

  data () {
    return {
      now: new Date(),
      vertical:false,
      legendKey:new Date().getTime()
    }
  },
  mounted() {
    // Register an event listener when the Vue component is ready
    window.addEventListener('resize', this.onResize)
    if (/Android|webOS|iPhone|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent) || $(window).width() < 480 ){
      this.vertical = true
    }
  },

  beforeDestroy() {
    // Unregister the event listener before destroying this Vue instance
    window.removeEventListener('resize', this.onResize)
  },
  computed: {
    svgID () {
      return '_' + Math.random().toString(36).substr(2, 9);
    },
    isMobile() {
      return /Android|webOS|iPhone|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent)
    },
    position () {
      return this.vertical ? 'vertical' : 'horizontal'
    },
    tooltipTransform () {
      return `translate(${this.tooltipX}, ${this.tooltipY})`
    },
    heatmap () {
      return new Heatmap(this.endDate, this.values, this.max)
    },
    width () {
      return {
        horizontal: this.LEFT_SECTION_WIDTH + (this.SQUARE_SIZE * this.heatmap.weekCount) + this.SQUARE_BORDER_SIZE,
        vertical: this.LEFT_SECTION_WIDTH + (this.SQUARE_SIZE * DAYS_IN_WEEK) + this.RIGHT_SECTION_WIDTH
      }
    },
    heigth () {
      return {
        horizontal: this.TOP_SECTION_HEIGTH + (this.SQUARE_SIZE * DAYS_IN_WEEK) + this.SQUARE_BORDER_SIZE + this.BOTTOM_SECTION_HEIGTH,
        vertical: this.TOP_SECTION_HEIGTH + (this.SQUARE_SIZE * this.heatmap.weekCount) + this.SQUARE_BORDER_SIZE
      }
    },
    viewbox () {
      return `0 0 ${this.width[this.position]} ${this.heigth[this.position]}`
    },
    daysLabelWrapperTransform () {
      return {
        horizontal: `translate(0, ${this.TOP_SECTION_HEIGTH})`,
        vertical: `translate(${this.LEFT_SECTION_WIDTH}, 0)`
      }
    },
    monthsLabelWrapperTransform () {
      return {
        horizontal: `translate(${this.LEFT_SECTION_WIDTH}, 0)`,
        vertical: `translate(0, ${this.TOP_SECTION_HEIGTH})`
      }
    },
    legendWrapperTransform () {
      return {
        horizontal: `translate(30, ${this.heigth[this.position] - this.BOTTOM_SECTION_HEIGTH})`,
        vertical: `translate(${this.LEFT_SECTION_WIDTH + (this.SQUARE_SIZE * DAYS_IN_WEEK)}, ${this.TOP_SECTION_HEIGTH})`
      }
    },
    yearWrapperTransform () {
      return `translate(${this.LEFT_SECTION_WIDTH}, ${this.TOP_SECTION_HEIGTH})`
    },

    SQUARE_BORDER_SIZE: () => SQUARE_SIZE / 5,
    SQUARE_SIZE () { return SQUARE_SIZE + this.SQUARE_BORDER_SIZE },
    TOP_SECTION_HEIGTH () { return SQUARE_SIZE + (SQUARE_SIZE / 2) },
    RIGHT_SECTION_WIDTH () { return this.SQUARE_SIZE * 3 },
    BOTTOM_SECTION_HEIGTH () { return SQUARE_SIZE + (SQUARE_SIZE / 2) },
    LEFT_SECTION_WIDTH () { return Math.ceil(SQUARE_SIZE * 2.5) },

    lo () {
      if (this.locale) {
        return {
          months: this.locale.months || DEFAULT_LOCALE.months,
          days: this.locale.days || DEFAULT_LOCALE.days,
          on: this.locale.on || DEFAULT_LOCALE.on,
          less: this.locale.less || DEFAULT_LOCALE.less,
          more: this.locale.more || DEFAULT_LOCALE.more
        }
      }
      return DEFAULT_LOCALE
    }
  },

  methods: {
    returnTooltipContent(day){
      if (day.count != null) {
        return JSON.stringify(day)
        // return `${day.count} ${this.tooltipUnit} ${this.lo.on} ${this.lo.months[day.date.getMonth()]} ${day.date.getDate()}, ${day.date.getFullYear()}`
      }else if (this.noDataText) {
        return `${this.noDataText}: ${this.lo.months[day.date.getMonth()]} ${day.date.getDate()}, ${day.date.getFullYear()} `
      }
    },
    tooltipOptions (day) {
      if (this.tooltip) {
        if (day.count != null) {
          return {
            content: JSON.stringify(day),
            delay: { show: 150, hide: 50 },
            // defaultTrigger: window.innerWidth > 768 ? 'hover focus click' : 'click'
          }
        } else if (this.noDataText) {
          return {
            content: `${this.noDataText}: ${this.lo.months[day.date.getMonth()]} ${day.date.getDate()}, ${day.date.getFullYear()}`,
            delay: { show: 150, hide: 50 },
            // defaultTrigger: window.innerWidth > 768 ? 'hover focus click' : 'click'
          }
        }
      }
      return false
    },
    getWeekPosition (index) {
      if (this.vertical) {
        return `translate(0, ${(this.SQUARE_SIZE * this.heatmap.weekCount) - ((index + 1) * this.SQUARE_SIZE)})`
      }
      return `translate(${index * this.SQUARE_SIZE}, 0)`
    },
    getDayPosition (index) {
      if (this.vertical) {
        return `translate(${index * this.SQUARE_SIZE}, 0)`
      }
      return `translate(0, ${index * this.SQUARE_SIZE})`
    },
    getMonthLabelPostion (month) {
      let position = { x: 0, y: 0 }
      position.x = this.vertical ? 3 : this.SQUARE_SIZE * month.index
      position.y = this.vertical ? (this.SQUARE_SIZE * this.heatmap.weekCount) - (this.SQUARE_SIZE * (month.index)) - (this.SQUARE_SIZE / 4) : this.SQUARE_SIZE - this.SQUARE_BORDER_SIZE
      return position
    },
    getLegendRectVerticalXInPosition(index){
      var finalX = 0;
      if (index === 0) {
        return finalX
      }
      var previousTextWidth = this.getTextWidth(this.heatmap.getColorValue(index - 1),"11px Source Sans Pro,-apple-system,BlinkMacSystemFont,Segoe UI,Roboto,Helvetica Neue,Arial,Noto Sans,sans-serif,Apple Color Emoji,Segoe UI Emoji,Segoe UI Symbol,Noto Color Emoji");
      finalX += (SQUARE_SIZE + previousTextWidth + 10)
      finalX += this.getLegendRectVerticalXInPosition(index - 1)
      return finalX
    },
    onResize(event) {
      var currentVertical = this.vertical;
      if (/Android|webOS|iPhone|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent) || $(window).width() < 480 ){
        this.vertical = true
      }else{
        this.vertical = false
      }
      if (currentVertical != this.vertical){
        this.$forceUpdate();
      }
    },
    getTempCanvas() {
      if (this.tempCanvas) {
        return this.tempCanvas
      }
      this.tempCanvas = document.createElement('canvas');
      return this.tempCanvas;
    },
    getTextWidth(text, font) {
      const canvas = this.getTempCanvas()
      const context = canvas.getContext('2d');

      context.font = font || getComputedStyle(document.body).font;

      return Math.ceil(context.measureText(text).width);
    },
  }
}
</script>

<style scoped>
svg.vch__wrapper {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  line-height: 10px;
}

svg.vch__wrapper .vch__months__labels__wrapper text.vch__month__label {
  font-size: 10px;
}

svg.vch__wrapper .vch__days__labels__wrapper text.vch__day__label,
svg.vch__wrapper .vch__legend__wrapper text {
  font-size: 9px;
}

svg.vch__wrapper .vch__months__labels__wrapper text.vch__month__label,
svg.vch__wrapper .vch__days__labels__wrapper text.vch__day__label,
svg.vch__wrapper .vch__legend__wrapper text {
  fill: #767676;
}

svg.vch__wrapper rect.vch__day__square:hover {
  stroke: #555;
  stroke-width: 1px;
}

svg.vch__wrapper rect.vch__day__square:focus {
  outline: none;
}
</style>

<style>
.vue-tooltip-theme.tooltip {
  display: block !important;
  z-index: 10000;
}

.vue-tooltip-theme.tooltip .tooltip-inner {
  background: rgba(0, 0, 0, .7);
  border-radius: 3px;
  color: #ebedf0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  font-size: 12px;
  line-height: 16px;
  padding: 14px 10px;
}

.vue-tooltip-theme.tooltip .tooltip-inner b {
  color: white;
}

.vue-tooltip-theme.tooltip .tooltip-arrow {
  width: 0;
  height: 0;
  border-style: solid;
  position: absolute;
  margin: 5px;
  border-color: black;
  z-index: 1;
}

.vue-tooltip-theme.tooltip[x-placement^="top"] {
  margin-bottom: 5px;
}

.vue-tooltip-theme.tooltip[x-placement^="top"] .tooltip-arrow {
  border-width: 5px 5px 0 5px;
  border-left-color: transparent !important;
  border-right-color: transparent !important;
  border-bottom-color: transparent !important;
  bottom: -5px;
  left: calc(50% - 5px);
  margin-top: 0;
  margin-bottom: 0;
}

.vue-tooltip-theme.tooltip[x-placement^="bottom"] {
  margin-top: 5px;
}

.vue-tooltip-theme.tooltip[x-placement^="bottom"] .tooltip-arrow {
  border-width: 0 5px 5px 5px;
  border-left-color: transparent !important;
  border-right-color: transparent !important;
  border-top-color: transparent !important;
  top: -5px;
  left: calc(50% - 5px);
  margin-top: 0;
  margin-bottom: 0;
}

.vue-tooltip-theme.tooltip[x-placement^="right"] {
  margin-left: 5px;
}

.vue-tooltip-theme.tooltip[x-placement^="right"] .tooltip-arrow {
  border-width: 5px 5px 5px 0;
  border-left-color: transparent !important;
  border-top-color: transparent !important;
  border-bottom-color: transparent !important;
  left: -5px;
  top: calc(50% - 5px);
  margin-left: 0;
  margin-right: 0;
}

.vue-tooltip-theme.tooltip[x-placement^="left"] {
  margin-right: 5px;
}

.vue-tooltip-theme.tooltip[x-placement^="left"] .tooltip-arrow {
  border-width: 5px 0 5px 5px;
  border-top-color: transparent !important;
  border-right-color: transparent !important;
  border-bottom-color: transparent !important;
  right: -5px;
  top: calc(50% - 5px);
  margin-left: 0;
  margin-right: 0;
}

.vue-tooltip-theme.tooltip[aria-hidden='true'] {
  visibility: hidden;
  opacity: 0;
  transition: opacity .15s, visibility .15s;
}

.vue-tooltip-theme.tooltip[aria-hidden='false'] {
  visibility: visible;
  opacity: 1;
  transition: opacity .15s;
}
</style>
