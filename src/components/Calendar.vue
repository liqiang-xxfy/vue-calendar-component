<template>
    <div class="cal-comp"
        :class="{'light':theme1==='light','dark':theme1==='dark','mini':size==='mini','medium':size==='medium','small':size!=='medium'&&size!=='mini'}">
        <div class="yearMonth">
            <div class="year">
                {{monthLang[viewYMD.monthIndex]}} {{viewYMD.year}}
            </div>
            <div class="month">
                <div class="arrow-back" @click="jumpMonth(-1)">
                    <svg class="icon" width=100% height=100% viewBox="0 0 1024 1024" version="1.1"
                        xmlns="http://www.w3.org/2000/svg">
                        <path :fill="iconFill"
                            d="M377.052007 512l386.3073-388.203486L705.12909 65.290005 318.845327 453.493491l-58.20668 58.506509 58.20668 58.505485 386.283764 388.203486 58.230216-58.506509L377.052007 512zM377.052007 512" />
                    </svg>
                </div>
                <div class="today" @click="selectedToDay">
                    <span>{{today}}</span>
                </div>
                <div class="arrow-to" @click="jumpMonth(1)">
                    <svg class=" icon" width=100% height=100% viewBox="0 0 1024 1024" version="1.1"
                        xmlns="http://www.w3.org/2000/svg">
                        <path :fill="iconFill"
                            d="M377.052007 512l386.3073-388.203486L705.12909 65.290005 318.845327 453.493491l-58.20668 58.506509 58.20668 58.505485 386.283764 388.203486 58.230216-58.506509L377.052007 512zM377.052007 512" />
                    </svg>
                </div>

            </div>
        </div>
        <div class="week">
            <div class="week-item" v-for="(item,index) in weekIndexArr" :key="item+index">
                {{weekLang[item]}}
            </div>
        </div>
        <div class="cell-day" @click="selectedDay">
            <div class="day-item"
                :class="{'today':currentYMD.year===item.year&&currentYMD.monthIndex===item.monthIndex&&currentYMD.day===item.day,
                'not-month':item.monthIndex!==viewYMD.monthIndex,
                'selected':selectedYMD.year===item.year&&selectedYMD.monthIndex===item.monthIndex&&selectedYMD.day===item.day}"
                v-for="(item,index) in visibleCalendar" :key="item+index" :data-index="index"
                :data-monthIndex="item.monthIndex">
                {{item.day}}
            </div>
        </div>
    </div>

</template>

<script>
export default {
    name: 'calendar',
    components: {},
    props: {
        events: {
            type: Array,
            default: () => []
        },
        value: {
            type: [String, Date],
            default: () => new Date()
        },
        size: {
            type: String,
            default: "small" //medium/small/mini
        },
        sundayFirst: {
            type: Boolean,
            default: true
        },
        theme: {
            type: String,
            default: "dark"
        },
        lang: {
            type: String,
            default: "en"
        }
    },
    data() {
        return {
            viewYMD: { year: -1, monthIndex: -1, day: -1 },
            // currentYMD: { year: "", monthIndex: 12, day: 7 },
            theme1: "",
            //#27292A

        }
    },
    watch: {},
    computed: {
        hasEvent() {
            return this.events !== [];
        },
        currentYMD() {
            let date = new Date();
            return this.getYearMonthDay(date);
        },
        selectedYMD() {
            let date = new Date(this.value);

            if (date.toString() === "Invalid Date") {
                date = new Date();
            } else {
                let { year } = this.getYearMonthDay(date)
                if (year < 1901 || year > 2050) {
                    date = new Date();
                }
            }
            return this.getYearMonthDay(date);
        },
        weekIndexArr() {
            return this.sundayFirst ? [0, 1, 2, 3, 4, 5, 6] : [1, 2, 3, 4, 5, 6, 0];
        },
        visibleCalendar() {
            let calendatArr = [];
            //先得到当前的年，月，日

            let { year, monthIndex, day } = this.viewYMD;

            if (year < 0) return calendatArr;

            //获取当月的第一天 得到2019 - 5 - 1
            let currentFirstDay = new Date(year, monthIndex, 1);

            console.log("currentFirstDay:", currentFirstDay);

            //获取第一天是星期几 得到 3
            let weekIndex = currentFirstDay.getDay();

            console.log("weekIndex:", weekIndex);

            //需要显示上个月的天数,适应以周一或周日开始
            let lastDay = weekIndex - (this.sundayFirst ? 0 : 1);
            console.log("lastDay:", lastDay);


            if (lastDay < 0) lastDay += 7;

            const oneDay = 24 * 60 * 60 * 1000;

            //用当月的第一天减去 周几前面几天 这样就能得到上个月开始的天数 （当前月1号是周三，那么周一就是上个月的最后两天）
            let startTime = currentFirstDay.getTime() - lastDay * oneDay;
            console.log("startTime:", startTime);

            //然后得到所有的日期
            for (let i = 0; i < 42; i++) {
                calendatArr.push(this.getYearMonthDay(new Date(startTime + i * oneDay)));
            };
            return calendatArr;
        },
        weekLang() {
            if (this.lang === "zh") {
                return ["日", "一", "二", "三", "四", "五", "六", ""];
            } else return ["Su", "Mo", "Tu", "We", "Th", "Fr", "Sa", ""]
        },
        monthLang() {
            if (this.lang === "zh") {
                return ["一月", "二月", "三月", "四月", "五月", "六月", "七月", "八月", "九月", "十月", "十一月", "十二月", ""];
            } else return ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sept", "Oct", "Nov", "Dec", ""]
        },
        today() {
            if (this.lang === "zh") {
                return "今天"
            } else return "Today"
        },
        iconFill() {
            if (this.theme === "light") {
                return "#dbdbdb";
            } else return "#dbdbdb";
        }

    },
    methods: {
        getYearMonthDay(date) {
            let year = date.getFullYear();
            let monthIndex = date.getMonth();
            let day = date.getDate();
            return { year, monthIndex, day };
        },
        getDate(YMD) {
            return YMD.year.toString() + "-" + (YMD.monthIndex + 1) + "-" + YMD.day;
        },
        jumpMonth(i) {
            let { year, monthIndex, day } = this.viewYMD
            if ((monthIndex + i) > 11) {
                monthIndex = monthIndex + i - 12;
                year++;
            } else if ((monthIndex + i) < 0) {
                monthIndex = monthIndex + i + 12;
                year--;
            } else {
                monthIndex = monthIndex + i;
            }

            if (year < 1901 || year > 2050 || monthIndex < 0 || day < 0) return;

            this.viewYMD = { year, monthIndex, day }
        },
        selectedDay(e) {
            let index = e.target.getAttribute('data-index');
            let monthIndex = e.target.getAttribute('data-monthIndex');
            if (index) {
                let selectedYMD = this.visibleCalendar[index];

                if (monthIndex == this.viewYMD.monthIndex) {
                    this.$emit('input', this.getDate(selectedYMD));
                }
                this.viewYMD = selectedYMD;
            }
            // item.monthIndex!==viewYMD.monthIndex

        },
        selectedToDay() {
            let date = new Date();
            this.$emit('input', this.getDate(this.getYearMonthDay(date)))
            this.viewYMD = this.getYearMonthDay(date);
        }

    },
    beforeCreate() { },
    created() { },
    beforeMount() {


    },
    mounted() {
        // let date = new Date();
        this.viewYMD = this.selectedYMD;
        // if (typeof this.value === "string") {
        //     this.$emit('input', date.getFullYear().toString() + "-" + (date.getMonth() + 1) + "-" + date.getDate());
        // }
        console.log("visibleCalendar:", this.visibleCalendar)

    },
    beforeUpdate() { },
    updated() { },
    activated() { },
    deactivated() { },
    beforeDestroy() { },
    destroyed() { },
    errorCaptured() { }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less" scoped>
// .dsasda {
//     $color: black !global;
// }0A84FF
.theme-color(@back:#27292a;@font-1:#c0bdbd;@font-2:#a3a3a7;@font-3:#757677;@item-back:#434344;@selected:#797c7e;@today:#0a84ff) {
    background-color: @back;

    .yearMonth {
        color: @font-1;
        .month {
            .arrow-back,
            .arrow-to {
                &:hover {
                    box-shadow: 0px 0px 2px 1px #797c7e8c inset;
                }
            }
        }
    }
    .week {
        color: @font-3;
    }
    .cell-day {
        .day-item {
            color: @font-2;
            background: @item-back;
            &.not-month {
                color: @font-3;
            }
            &:hover {
                background: @selected;
            }
            &.selected {
                background: none;
                // outline: 2px solid @selected;
                box-shadow: 0px 0px 1px 2px @selected inset;

                // color: white;
            }
            &.today {
                background: @today;
                color: white;
                box-shadow: none;
            }
        }
    }
}
.theme-size(@width:350px;@header-height:40px;@font-header:19px;@font-content:16px;@item-padding:2.2%;@radius:5px;@header-padding:12px;) {
    width: @width;
    font-size: @font-content;
    .yearMonth {
        font-size: @font-header;
        height: @header-height;
        .year {
            padding-left: @header-padding;
        }
        .month {
            .arrow-back {
                max-width: @font-header;
            }
            .today {
                padding: 0 @font-header;
            }
            .arrow-to {
                max-width: @font-header;
            }
        }
    }
    .cell-day {
        .day-item {
            padding: @item-padding 0;
            border-radius: @radius;
        }
    }
}
.cal-comp {
    .theme-color();
    &.mini {
        .theme-size(@width:200px;@header-height:30px;@font-header:15px;@font-content:13px;@item-padding:1.3%;@radius:4px;@header-padding:5px;);
    }
    &.small {
        .theme-size();
    }
    &.medium {
        .theme-size(@width:500px;@header-height:50px;@font-header:25px;@font-content:20px;@item-padding:2.7%;@radius:5px;@header-padding:22px;);
    }
    border-radius: 10px;
    padding: 8px;

    // display: flex;
    // flex-direction:column;
    .yearMonth {
        width: 100%;

        display: flex;
        // justify-content: space-between;
        align-items: center;
        .year {
            width: 40%;
            text-align: left;
            flex-grow: 2;
            line-height: 100%;
        }
        .month {
            user-select: none;

            flex-grow: 2;

            display: flex;
            // align-items: center;
            justify-content: center;
            .arrow-back {
                min-width: 15px;
                cursor: pointer;
                border-radius: 4px;
            }
            .today {
                display: flex;
                align-items: center;
                line-height: 100%;
                cursor: pointer;
            }
            .arrow-to {
                min-width: 15px;

                transform: rotateY(180deg);
                cursor: pointer;
                border-radius: 4px;
            }
            // padding-right: 8%;
        }
    }
    .week {
        user-select: none;

        display: flex;
        justify-content: space-around;
        user-select: none;
        .week-item {
            width: 12.6%;
            margin: 0.8% 0.2%;
            padding: 1% 0;
        }
    }
    .cell-day {
        // height: 700px;
        user-select: none;

        display: flex;
        flex-wrap: wrap;
        justify-content: space-around;
        align-content: flex-start;
        .day-item {
            width: 12.6%;
            margin: 0.8% 0.2%;
            cursor: pointer;
            // box-sizing: border-box;

            &.not-month {
                background: none;
                opacity: 0.4;
            }
        }
    }
}
</style>
