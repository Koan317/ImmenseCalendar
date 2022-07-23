<!-- 
    Author: Koan
    Version: 1.0
    Date: July 23rd 2022
    Description: an immense calendar support selecting days one by one
 -->
<template>
    <div class="total-width">
        <div class="display-flex">
            <div v-if="optional.lang == 'EN'">
                <el-checkbox :indeterminate="isIndeterminate" v-model="selectAll" @change="changeSelectAll"
                    class="display-block">Select All
                </el-checkbox>
                <div v-for="(day, index_en) in DAYS_EN" :key="index_en" class="day-title">{{ day
                }}</div>
            </div>
            <div v-else-if="optional.lang == 'JP'">
                <el-checkbox :indeterminate="isIndeterminate" v-model="selectAll" @change="changeSelectAll"
                    class="display-block">すべて選択
                </el-checkbox>
                <div v-for="(day, index_jp) in DAYS_JP" :key="index_jp" class="day-title">{{ day
                }}</div>
            </div>
            <div v-else-if="optional.lang == 'KO'">
                <el-checkbox :indeterminate="isIndeterminate" v-model="selectAll" @change="changeSelectAll"
                    class="display-block">모두 선택
                </el-checkbox>
                <div v-for="(day, index_ko) in DAYS_KO" :key="index_ko" class="day-title">{{ day
                }}</div>
            </div>
            <div v-else>
                <el-checkbox :indeterminate="isIndeterminate" v-model="selectAll" @change="changeSelectAll"
                    class="display-block">全选
                </el-checkbox>
                <div v-for="(day, index_cn) in DAYS_CN" :key="index_cn" class="day-title">{{ day
                }}</div>
            </div>
        </div>
        <el-checkbox-group v-model="optional.selectedDays" class="button-row" :disabled="optional.disabled"
            @change="changeDays">
            <el-checkbox-button v-for="(day, index_day) in daysInMonth" :key="index_day" :label="day" class="button-day"
                :disabled="disabledDate(optional.year, optional.month, day)">
                {{ day }}
            </el-checkbox-button>
        </el-checkbox-group>
    </div>
</template>

<script>
/**
 * custom variable
 * @param {number} year - the year to show
 * @param {number} month - the month to show
 * @param {string} lang - interface language, 'EN','JP','KO' are available, and other values lead to Chinese
 * @param {array} selectedDays - selectedDays, begin from 1, both number array and string array are available
 * @param {number} buttonWidth - each width of a day button
 * @param {boolean} disabled - if is disabled, true lead to disabled and other values lead to false
 * @function onChange - return selected days list
 * @function disabledDays - use callback funtion to mark disabled days, need to return a boolean value
 */
const DAYS_CN = ['日', '一', '二', '三', '四', '五', '六'];
const DAYS_EN = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
const DAYS_JP = ['日', '月', '火', '水', '木', '金', '土'];
const DAYS_KO = ['일', '월', '화', '수', '목', '금', '토'];
export default {
    props: {
        year: {
            type: [Number, String],
            default: new Date().getFullYear()
        },
        month: {
            type: [Number, String],
            default: new Date().getMonth()
        },
        lang: {
            type: String,
            default: 'CN'
        },
        selectedDays: {
            type: Array,
            default: () => { return []; }
        },
        buttonWidth: {
            type: [Number, String],
            default: 120
        },
        disabled: {
            type: [Boolean, String],
            default: false
        }
    },
    data() {
        return {
            DAYS_CN,
            DAYS_EN,
            DAYS_JP,
            DAYS_KO,
            daysInMonth: 0,
            firstDayOfMonth: 0,
            optional: {
                year: undefined,
                month: undefined,
                buttonWidth: undefined,
                disabled: undefined,
                selectedDays: [],
                lang: undefined
            },
            isIndeterminate: false,
            selectAll: false
        };
    },
    created() {
        this.optional.year = parseInt(this.year);
        this.optional.month = parseInt(this.month);
        this.optional.buttonWidth = parseInt(this.buttonWidth);
        this.optional.disabled = this.disabled == 'true';
        if (this.selectedDays) {
            this.selectedDays.forEach(day => {
                this.optional.selectedDays.push(parseInt(day));
            });
        }
        this.optional.lang = this.lang.toString();
        this.getDaysInMonth(this.optional.year, this.optional.month);
    },
    mounted() {
        this.$nextTick(() => {
            for (let ele of document.getElementsByClassName("day-title")) {
                ele.setAttribute("style", "width:" + this.optional.buttonWidth + "px;");
            }
            for (let ele of document.getElementsByClassName("button-day")) {
                ele.setAttribute("style", "width:" + this.optional.buttonWidth + "px;");
            }
            document.getElementsByClassName("total-width")[0].setAttribute("style", "width:" + (7 * this.optional.buttonWidth + 154) + "px;");
            let firstButtonNode = document.getElementsByClassName("button-day")[0];
            firstButtonNode.setAttribute("style", firstButtonNode.getAttribute("style") + "margin-left:" + (this.firstDayOfMonth * (this.optional.buttonWidth + 22) + 10) + "px;");
        });
        this.changeDays(this.optional.selectedDays);
    },
    methods: {
        /**
         * return number of days in a month
         * @param {number} year - the year to show
         * @param {number} month - the month to show, from 0 to 11
         */
        getDaysInMonth(year, month) {
            let day = year && month ? new Date(year, month, 1) : new Date();
            day.setMonth(day.getMonth() + 1);
            day.setDate(0);
            this.daysInMonth = day.getDate();
            day.setDate(1);
            this.firstDayOfMonth = day.getDay();
        },
        /**
         * return the selected days to the parent component
         * @param {array} event - click event, including the array of selected days which begin from 1
         */
        changeDays(event) {
            let eles = document.getElementsByClassName("button-day");
            let availableDays = [];
            for (let index = 0; index < eles.length; index++) {
                if (!eles[index].getAttribute("aria-disabled")) {
                    availableDays.push(index + 1);
                }
            }
            event.sort((a, b) => { return a - b; });
            if (event.length == 0) {
                this.isIndeterminate = false;
                this.selectAll = false;
            } else if (event.toString() == availableDays.toString()) {
                this.isIndeterminate = false;
                this.selectAll = true;
            } else {
                this.isIndeterminate = true;
                this.selectAll = false;
            }
            let selectedDays = Array.from(event);
            this.$emit('onChange', selectedDays);
        },
        /**
         * judge current day is able or disable
         * @param {number} year - the year to show
         * @param {number} month - the month to show
         * @param {number} date - the date to show
         * @returns {boolean} is the day available or not
         */
        disabledDate(year, month, date) {
            let isDisabled = false;
            this.$emit('disabledDays', new Date(year, month, date), val => { isDisabled = val });
            return isDisabled;
        },
        /**
         * deal with logic about select all days
         * @param {boolean} isAll - is select all or unselect all
         */
        changeSelectAll(isAll) {
            this.isIndeterminate = false;
            let eles = document.getElementsByClassName("button-day");
            if (isAll) {
                for (let index = 0; index < eles.length; index++) {
                    if (!eles[index].getAttribute("aria-disabled")) {
                        this.optional.selectedDays.push(index + 1);
                    }
                }
            } else {
                this.optional.selectedDays = [];
            }
        }
    }
};
</script>

<style scoped lang="css">
.display-flex {
    display: flex;
}

.display-block {
    display: block;
}

.button-row {
    display: flex;
    flex-wrap: wrap;
}

.day-title {
    display: inline-block;
    min-width: 60px;
    border: 1px solid #00000000;
    margin: 10px 10px;
    text-align: center;
}

.button-day {
    margin: 10px 10px;
    min-width: 60px;
    text-align: center;
    border: 1px solid #ddd;
    border-radius: 4px;
    cursor: pointer;
}

.el-checkbox-button {
    background-color: #fff;
    transition: all 0.5s linear;
}

.el-checkbox-button.is-checked {
    background-color: #409EFF;
    border: 1px solid #409EFF;
    color: white;
    transition: all 0.5s linear;
}

.el-checkbox-button.is-checked:hover {
    animation: button-checking 0.5s;
    background-color: #409EFF;
}

.el-checkbox-button:hover {
    animation: button-hover 0.5s;
    background-color: #ccc;
}

.button-day>>>.el-checkbox-button__inner {
    width: 100%;
    height: 100%;
    border: 0;
    border-radius: 4px;
    background-color: #00000000;
    box-shadow: none;
}

.button-day>>>.el-checkbox-button__inner:hover {
    color: black;
}


@keyframes button-hover {
    0% {
        background: #fff;
        color: #606266;
    }

    100% {
        background: #ccc;
        color: black;
    }
}

@keyframes button-checking {
    0% {
        background: #409EFF;
        color: white;
    }

    100% {
        background: #409EFF;
        color: white;
    }
}
</style>
