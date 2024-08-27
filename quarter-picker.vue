<template>
    <el-popover
      trigger="click"
      v-model="pickerVisible"
      :disabled="disabled"
      placement="bottom-start"
      transition="el-zoom-in-top"
    >
        <el-input 
          ref="reference"
          slot="reference"
          class="el-date-editor"
          readonly
          :disabled="disabled"
          :size="size"
          :placeholder="placeholder"
          :value="displayValue"
          :validate-event="false"
          :style="{ width }"
          @mouseenter.native="handleMouseEnter"
          @mouseleave.native="showClose = false"
        >
            <i slot="prefix" class="el-input__icon" :class="triggerClass"></i>
            <i 
              slot="suffix"
              class="el-input__icon"
              :class="[showClose ? '' + clearIcon : '']"
              @click="handleClickIcon"
              @mousedown="handleMousedownIcon"
            >
            </i>
        </el-input>
            <div class="el-date-picker">
                <div class="el-picker-panel__body">
                    <div class="el-date-picker__header el-date-picker__header--bordered" style="margin:0px; line-height:30px">
                        <button type="button" aria-label="前一年"
                            class="el-picker-panel__icon-btn el-date-picker__prev-btn el-icon-d-arrow-left" @click="prevYear">
                        </button>
                        <span role="button" class="el-date-picker__header-label">{{ yearLabel }}</span>
                        <button type="button" aria-label="后一年"
                            class="el-picker-panel__icon-btn el-date-picker__next-btn el-icon-d-arrow-right" @click="nextYear">
                        </button>
                    </div>
                    <div class="el-picker-panel__content">
                        <table class="el-quarter-table" @click="handleTableClick">
                            <tbody>
                                <tr>
                                    <td class="available" :class="getCellStyle(0)">
                                            <a class="cell">第一季度</a>
                                    </td>
                                    <td class="available" :class="getCellStyle(1)">
                                            <a class="cell">第二季度</a>
                                    </td>
                                </tr>
                                <tr>
                                    <td class="available" :class="getCellStyle(2)">
                                            <a class="cell">第三季度</a>
                                    </td>
                                    <td class="available" :class="getCellStyle(3)">
                                            <a class="cell">第四季度</a>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
    </el-popover>
</template>

<script>
import { formatDate, prevYear, nextYear, range, nextDate, isDateObject, parseDate } from 'element-ui/src/utils/date-util'
import { hasClass } from 'element-ui/src/utils/dom'

// 获取指定年份和季度的所有日期
const datesInYearAndQuarter = (year, quarter) => {
    const numOfDays = getDayCountOfQuarter(year, quarter)
    const firstDay = new Date(year, quarter * 3, 1)
    return range(numOfDays).map(n => nextDate(firstDay, n))
}

// 获取指定年份和季度总天数
const getDayCountOfQuarter = (year, quarter) => {
    switch (quarter) {
        case 0: // 第一季度包含二月，需要对是否闰年进行判断处理
            if (year % 4 === 0 && year % 100 !== 0 || year % 400 === 0) {
                return 91
            } else {
                return 90
            }
        case 1:
            return 91
        default:
            return 92
    }
}

export default {
    name: 'QuarterPicker',
    props: {
        size: String,
        format: String,
        valueFormat: String,
        placeholder: String,
        prefixIcon: String,
        clearIcon: {
            type: String,
            default: 'el-icon-circle-close'
        },
        disabled: Boolean,
        clearable: {
            type: Boolean,
            default: true
        },
        width: {
            type: String,
            default: ''
        },
        disabledDate: {},
        value: null
    },
    data() {
        return {
            showClose: false,
            pickerVisible: false,
            date: new Date(),
            quarterText: ['一', '二', '三', '四']
        }
    },
    computed: {
        triggerClass() {
            return this.prefixIcon || 'el-icon-date'
        },
        displayValue() {
            if (!this.value) return null
            // 季度，从0开始
            const quarter = parseInt(this.parsedValue.getMonth() / 3)
            let fDate = formatDate(this.parsedValue, this.format)
            fDate = fDate.replace(/q/, quarter + 1).replace(/Q/, this.quarterText[quarter])
            return fDate
        },
        year() {
            return this.date.getFullYear()
        },
        yearLabel() {
            return this.year + ' 年'
        },
        parsedValue() {
            if (!this.value) {
                return this.value
            }
            if (isDateObject(this.value)) {
                return this.value
            }
            // 非时间格式且设置了valueFormat，进行时间转换
            if (this.valueFormat) {
                return parseDate(this.value, this.valueFormat)
            }
            // 非时间格式且未设置valueFormat，再尝试转换时间
            return new Date(this.value)
        }
    },
    watch: {
        value(value) {
            this.date == value ? this.parsedValue : new Date()
        }
    },
    methods: {
        handleMouseEnter() {
            if (!this.disabled && this.value && this.clearable) {
                this.showClose = true
            }
        },
        handleClickIcon(event) {
            if (!this.disabled && this.showClose) {
                this.$emit('input', null)
                this.$emit('change', null)
                this.showClose = false
                this.pickerVisible = false
                this.$refs.reference.blur()
            }
        },
        handleMousedownIcon(event) {
            // 阻止鼠标按下清空按钮，防止清空数据时季度选择面板闪现
            event.preventDefault()
        },
        handleTableClick(event) {
            let target = event.target
            if (target.tagName === 'A') {
                target = target.parentNode
            }
            if (target.tagName !== 'TD' || hasClass(target, 'disabled')) return
            
            const column = target.cellIndex
            const row = target.parentNode.rowIndex
            // 季度，从0开始
            const quarter = row * 2 + column
            // 季度开始月份，从0开始
            const month = quarter * 3
            let newDate = new Date(this.year, month, 1)
            if (this.valueFormat) {
                newDate = formatDate(newDate, this.valueFormat)
            }
            this.pickerVisible = false
            this.$emit('input', newDate)
            this.$emit('change', newDate)
        },
        prevYear() {
            this.date = prevYear(this.date)
        },
        nextYear() {
            this.date = nextYear(this.date)
        },
        getCellStyle(quarter) {
            const style = {}
            const today = new Date()
            const date = this.parsedValue ? this.parsedValue : today
            style.disabled = typeof this.disabledDate === 'function' ? datesInYearAndQuarter(this.year, quarter).every(this.disabledDate) : false
            // 当前选中的季度样式
            style.current = date.getFullYear() === this.year && parseInt(date.getMonth() / 3) === quarter
            // 今日所在季度样式
            style.quarter = today.getFullYear() === this.year && parseInt(today.getMonth() / 3) === quarter
            return style
        }
    }
}
</script>

<style>
.el-popper .popper__arrow {
    left: 35px !important;
}

.el-quarter-table {
    font-size: 16px;
    margin: -1px;
    border-collapse: collapse;
    width: 100%;
}

.el-quarter-table td {
    text-align: center;
    padding: 10px 3px;
    cursor: pointer;
}

.el-quarter-table td .cell {
    height: 36px;
    display: block;
    line-height: 36px;
    color: #606266;
    margin: 0 auto;
    border-radius: 18px;
}

.el-quarter-table td .cell:hover {
    color: #1890ff;
}

.el-quarter-table td.current:not(.disabled) .cell {
    color: #409eff;
}

.el-quarter-table td.quarter .cell {
    color: #409eff;
    font-weight: 700;
}

.el-quarter-table td.disabled .cell {
    background-color: #F5F7FA;
    cursor: not-allowed;
    color: #C0C4CC;
}
</style>
