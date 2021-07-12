<template>
    <h2 style="text-align: center;"> Date Custom</h2>
    <!-- <span> List seleect date: </span>
    <span style="" v-for="item in selectDate" :key="item" >
         {{ item }} / 
    </span> -->

    <div class="main">
        <div class="controlHeader">
            <div class="controlHeaderText">
                <span> Điểm gửi </span>
            </div>
            <div class="controlHeaderBtn">
                <span> {{ currentMonthName }} </span>
                <button class="btnControl" @click="clickControlMonth('-')"> &lt; </button>
                <span> {{ blurDate }} </span>
                <button class="btnControl"  @click="clickControlMonth('+')"> &gt; </button>
            </div>
        </div>
        <div style="clear: both"></div>
        <div class="calHeader">
            <div class="itemHeader"> <span> Mon </span> </div>
            <div class="itemHeader"> <span> Tue </span> </div>
            <div class="itemHeader"> <span> Wed </span> </div>
            <div class="itemHeader"> <span> Thu </span> </div>
            <div class="itemHeader"> <span> Fri </span> </div>
            <div class="itemHeader"> <span> Sat </span> </div>
             <div class="itemHeader"> <span> Sun </span> </div>
        </div>

        <div id="boxId" class="box" v-on:scroll="handleScroll" ref="boxRef"> 
            <div v-for="item in listDate" :key="item.day" class="item" 
                :id="item.day + '_' + item.month + '_' + item.year"
                @mouseover="setBlurDate(item.day, item.month, item.year)"
            >
                <span 
                    :ref="item.day + '-' + item.month + '-' + item.year"
                    v-bind:class="item.active + ' ' + item.activeMonthClass"
                    @click="clickDateAction(item.day, item.month, item.year)"
                >
                    {{ item.day }}
                </span>


            </div>
        </div>
    </div>
</template>

<script>
import { defineComponent, ref } from 'vue'
const NUM_DATE_FETCH = 14

export default defineComponent({
    name: 'DateCustom',
    props: ["date", "month", "year", "activeDateList", "title"],
    data() {
        return {
            listDate: [], // List Date For Render HTML
            selectDate: [], // List Date Selected 
            beforeDateState: "",
            nextDateState: "",

            blurDate: "", // 6-2021
            currentMonthName: "", // July
            checkBlurChangeMonth: "", // Bin Var Check Blur Change Month
            titleData: 'init data title....'
        }
    },
    created () {
        window.addEventListener('scroll', this.handleScroll);
    },
    unmounted () {
        window.removeEventListener('scroll', this.handleScroll);
    },

    mounted() {
        console.log(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> Mounted", this.inputData)
        this.initApp()
    },
   
    methods: {
        reset () {
            Object.assign(this.$data, this.$options.data());
        },
        async clickControlMonth (type) {
            console.log(">>>>>> Click Control Month")
            const [mBlur, yBlur] = this.blurDate.split("-")
            
            const initSelectData =  [...this.selectDate, ...this.activeDateList]
            await this.reset()

            this.blurDate = `${mBlur}-${yBlur}`
            let  monthControl = (type == "+") ? this.getNextMonthYear(Number(mBlur) , Number(yBlur))[0] : this.getBeforeMonthYear(Number(mBlur) , Number(yBlur))[0];
            let yearControl = (type == "+") ? this.getNextMonthYear(Number(mBlur) , Number(yBlur))[1] : this.getBeforeMonthYear(Number(mBlur) , Number(yBlur))[1];
            
            const initData = {
                date: 1,
                month: monthControl,
                year: yearControl,
                initSelectData: initSelectData
            }
            this.initApp(initData)
        },
        initApp (option) {
            var date, month, year;
            const now = new Date()
            
            this.selectDate =  (option && option.month) ? [...new Set(option.initSelectData)] : [...this.selectDate, ...this.activeDateList];
            date = (option && option.month) ? option.date : now.getDate();
            month = (option && option.month) ? option.month : (now.getMonth() + 1);
            year = (option && option.month) ? option.year : now.getFullYear();

            this.currentMonthName = this.getDateMonth(date, month, year, false)
            if (this.date && this.month && this.year && !option) {
                date = Number(this.date)
                month = Number(this.month)
                year = Number(this.year)
            }

            this.blurDate = `${month}-${year}`;
            this.listDate = [{
                day: date,
                month: month,
                year: year,
                active: true,
                activeMonthClass: this.calClassActiveMonth(month, year)
            }]

            var firstDayOnMonth = new Date()
            firstDayOnMonth.setDate(1)
            firstDayOnMonth.setMonth(month - 1)
            firstDayOnMonth.setFullYear(year)

            const offDayHead = firstDayOnMonth.getDay() - 1
            const numDayThisMonth = this.numDaysInMonth(month, year)

            var timeOffHeadList = []
            var timeMainList    = []
            var timeNextList    = []

            // Cal List Day Main
            for (let ii = 1; ii <= numDayThisMonth; ii++) {
                timeMainList.push({
                    day: ii,
                    month,
                    year,
                    active: (this.selectDate.indexOf(`${ii}-${month}-${year}`) >= 0 ) ? 'active' : '',
                    activeMonthClass: this.calClassActiveMonth(month, year)
                })
            }

            // Cal List Day Head
            const [monthHeadListBefore, yearHeadListBefore] = this.getBeforeMonthYear(month, year)
            timeOffHeadList = this.getListDayEndOfMonth(offDayHead, monthHeadListBefore, yearHeadListBefore )
            this.beforeDateState = timeOffHeadList[0]
            
            // Cal List Next Day
            const numCellEndList = 4 * NUM_DATE_FETCH - Number(timeMainList.length) - Number(timeOffHeadList.length); // 42 + 14 = 4*NUM_DATE_FETCH
            timeNextList = this.getListDayStartOfMonth(numCellEndList, month, year)
            this.nextDateState = timeNextList.slice(-1)[0] 

            const fullDayListBox = [...timeOffHeadList , ...timeMainList, ...timeNextList]

            this.listDate = fullDayListBox;
            const arrDate = [];
            for (let i = 1; i <= this.listDate.length; i++) {
                const itemKeyBind = `${fullDayListBox[i-1]["day"]}-${fullDayListBox[i-1]["month"]}-${fullDayListBox[i-1]["year"]}`
                const isActiveClass = this.selectDate.indexOf(itemKeyBind);
                arrDate.push({
                    day: fullDayListBox[i-1]["day"],
                    month: fullDayListBox[i-1]["month"],
                    year: fullDayListBox[i-1]["year"],
                    active: (isActiveClass > -1) ? 'active' : '',
                    activeMonthClass: this.calClassActiveMonth(fullDayListBox[i-1]["month"], fullDayListBox[i-1]["year"])
                })
            }
            this.listDate = arrDate;
            setTimeout(function() {
                var div = document.getElementById("boxId");
                div.scrollTop = 66
            }, 10)
        },
        setBlurDate (date, month, year) {
            const top = this.$refs.boxRef.getBoundingClientRect().top;
            const topElement = this.$refs[`${date}-${month}-${year}`].getBoundingClientRect().top;
            const left = this.$refs.boxRef.getBoundingClientRect().left;
            const leftElement = this.$refs[`${date}-${month}-${year}`].getBoundingClientRect().left;

            const topCalculate = Math.round(((topElement - top) / 33) + 1);
            const leftCalculate = Math.round((((leftElement - left) - 2.5) / 50 ) + 1);

            const monthYearStr = `${month}-${year}`;
            this.blurDate = monthYearStr
            this.currentMonthName = this.getDateMonth(date, month, year, false)
            if (!this.checkBlurChangeMonth) this.checkBlurChangeMonth = monthYearStr;

            var state = ""

            if (topCalculate == 1 || topCalculate == 2) {
               const date12 = this.getNextDayWithAnyDay(date, month, year)
               state = `${date12[13].month}-${date12[13].year}`;
            }
            if (topCalculate == 5 || topCalculate == 6) {
               const date56 = this.getBeforeDayWithAnyDay(date, month, year)
               state = `${date56[0].month}-${date56[0].year}`;
            }
            if (topCalculate == 3 || topCalculate == 4) state = `${month}-${year}`

            if (this.checkBlurChangeMonth != state) {
                this.checkBlurChangeMonth = monthYearStr;
                const arrListDateByMonth = []
                this.listDate.map((item) => {
                    const itemMonthCurrent =  this.checkBlurChangeMonth.split("-")[0]
                    arrListDateByMonth.push({
                        ...item,
                        activeMonthClass : (`${item.month}-${item.year}` == state) ? "activeMonth" : "noActiveMonth"
                    })
                })
                this.listDate = arrListDateByMonth
            }

            
            //  if (isBlur) { 
            //     this.checkBlurChangeMonth = monthYearStr;

            //     const arrListDateByMonth = []
            //     this.listDate.map((item) => {
            //         const itemMonthCurrent =  this.checkBlurChangeMonth.split("-")[0]
            //         arrListDateByMonth.push({
            //             ...item,
            //             activeMonthClass : (monthYearStr == this.blurDate && item.month == itemMonthCurrent ) ? "activeMonth" : "noActiveMonth"
            //         })
            //     })
            //     this.listDate = arrListDateByMonth
            //  }

            // const monthYearStr = `${month}-${year}`;
            // this.blurDate = monthYearStr
            // this.currentMonthName = this.getDateMonth(date, month, year, false)

            // if (!this.checkBlurChangeMonth) this.checkBlurChangeMonth = monthYearStr

            // if (this.checkBlurChangeMonth != this.blurDate) {
            //     this.checkBlurChangeMonth = monthYearStr;
            //     const arrListDateByMonth = []
            //     this.listDate.map((item) => {
            //         const itemMonthCurrent =  this.checkBlurChangeMonth.split("-")[0]
            //         arrListDateByMonth.push({
            //             ...item,
            //             activeMonthClass : (monthYearStr == this.blurDate && item.month == itemMonthCurrent ) ? "activeMonth" : "noActiveMonth"
            //         })
            //     })
            //     this.listDate = arrListDateByMonth
            // }
        },
        handleScroll ({ target: { scrollTop, clientHeight, scrollHeight }}) {
            scrollTop = Math.ceil(scrollTop)
            const { day, month, year } = this.nextDateState;
            const dayBefore = this.beforeDateState.day
            const monthBefore = this.beforeDateState.month
            const yearBefore = this.beforeDateState.year

            if (scrollTop + clientHeight >= scrollHeight) {
                console.log(">>>>>>> Scroll Button")
                const nextDayList = this.getNextDayWithAnyDay(day, month, year);
                this.listDate = [...this.listDate, ...nextDayList]
                this.nextDateState = nextDayList.slice(-1)[0]
            }

            if (scrollTop == 0) {
                console.log(">>>>>>> Scroll Topppp..")
                const beforeDayList = this.getBeforeDayWithAnyDay(dayBefore, monthBefore, yearBefore);
                this.listDate = [ ...beforeDayList, ...this.listDate];
                this.beforeDateState = beforeDayList[0];

                var div = document.getElementById("boxId");
                div.scrollTop = 1;
            }
        },
    
        clickDateAction (day, month, year) {
            const selectDateFormat = `${day}-${month}-${year}`;
            if (!this.selectDate) this.selectDate = []
            const countDateExit = this.selectDate.indexOf(selectDateFormat);

            this.listDate.map(item => {
                const isActive = (item && item.day && item.day == day && item.month == month && item.year == year) ? true : false;
                if (isActive) item.active = (isActive && item.active) ? "" : "active";
            })

            if (countDateExit == -1) {
                 this.selectDate = [...new Set(this.selectDate), selectDateFormat]
            } else {
                const arrSelectDate = []
                this.selectDate.map((item) => {
                    if (item != selectDateFormat) arrSelectDate.push(item)
                })
                this.selectDate = arrSelectDate;
            }
        },

        /*****************************************************/
        /****************** Helper **************************/

        // Get Num day in month: 28, 29, 30, 31
        numDaysInMonth (month, year) {
            return new Date(year, month, 0).getDate();
        },

        // Get Before Month Year
        getBeforeMonthYear (month, year) {
            return (month == 1) ? [12, year - 1] : [(month -1), year]
        },
        // Get Next Month Year
        getNextMonthYear (month, year) {
            return (month == 12) ? [1, year + 1] : [(month +1), year]
        },

        // Get List Day End Of Month From Num 
        getListDayEndOfMonth (num, month, year) {
            var list = []
            var numDaysInMonth = new Date(year, month, 0).getDate(); // numDaysInMonth(month, year)
            var daySatrtCal = numDaysInMonth - num + 1 - 14;

            for (let i = daySatrtCal; i <= numDaysInMonth; i++) {
                const itemMonth = this.getBeforeMonthYear((month + 1), year)[0];
                const itemYear = this.getBeforeMonthYear((month + 1), year)[1];
                list.push({
                    day: i,
                    month: itemMonth,
                    year: itemYear,
                    active: (this.selectDate.indexOf(`${i}-${itemMonth}-${itemYear}`) >= 0 ) ? "active" : "",
                    activeMonthClass: this.calClassActiveMonth(itemMonth, itemYear)
                })
            }
            return list
        },
        
        // Get List Day From Month From Num
        getListDayStartOfMonth (num, month, year) {
            var list = []
            for (let i = 1; i <= (num + 14); i++) {
                const itemMonth = this.getNextMonthYear(month, year)[0];
                const itemYear = this.getNextMonthYear(month, year)[1];
                list.push({
                    day: i,
                    month: itemMonth,
                    year: itemYear,
                    active: (this.selectDate.indexOf(`${i}-${itemMonth}-${itemYear}`) >= 0 ) ? 'active' : '',
                     activeMonthClass: this.calClassActiveMonth(itemMonth, itemYear)
                })
            }
            return list
        },

        getNextDayWithAnyDay (day , month, year) {
            var timeInit = new Date()
            timeInit.setDate(day)
            timeInit.setMonth(month - 1)
            timeInit.setFullYear(year)

            const numDaysInMonth = this.numDaysInMonth(month, year)
            const [nextMonth, nextYear] = this.getNextMonthYear(month, year)

            let nextTime = timeInit
            nextTime.setDate(nextTime.getDate() + NUM_DATE_FETCH);

            const nextTimeDay = nextTime.getDate()
            const nextTimeMonth = nextTime.getMonth() + 1
            const nextTimeYear = nextTime.getFullYear()
            
            const listNextDay = []
            if (nextTimeMonth == month && day != numDaysInMonth) {
                for (let ii = (day + 1); ii <= (day + NUM_DATE_FETCH); ii++) {
                    listNextDay.push({
                        day: ii,
                        month,
                        year,
                        active: (this.selectDate.indexOf(`${ii}-${month}-${year}`) >= 0 ) ? 'active' : '',
                         activeMonthClass: this.calClassActiveMonth(month, year)
                    })
                }
            } else {
                for (let i = (day + 1); i <= numDaysInMonth; i++) {
                    listNextDay.push({
                        day: i,
                        month,
                        year,
                        active: (this.selectDate.indexOf(`${i}-${month}-${year}`) >= 0 ) ? 'active' : '',
                         activeMonthClass: this.calClassActiveMonth(month, year)
                    })
                }
                for (let ii = 1; ii <= nextTimeDay; ii++) {
                    let itemMonth = this.getNextMonthYear(month, year)[0]
                    let itemYear = this.getNextMonthYear(month, year)[1]
                    listNextDay.push({
                        day: ii,
                        month: itemMonth,
                        year: itemYear,
                        active: (this.selectDate.indexOf(`${ii}-${itemMonth}-${itemYear}`) >= 0 ) ? 'active' : '', 
                         activeMonthClass: this.calClassActiveMonth(itemMonth, itemYear)
                    })
                }
            }

            return listNextDay;
        },
        
        getBeforeDayWithAnyDay (day, month, year) {
            var timeInit = new Date()
            timeInit.setDate(day)
            timeInit.setMonth(month - 1)
            timeInit.setFullYear(year)

            const numDaysInMonth = this.numDaysInMonth(month, year)
            const numDaysInBeforeMonth = this.numDaysInMonth(this.getBeforeMonthYear(month, year)[0], year)

            const [beforeMonth, beforeYear] = this.getBeforeMonthYear(month, year)

            let beforeTime = timeInit
            beforeTime.setDate(beforeTime.getDate() - NUM_DATE_FETCH);

            const beforeTimeDay = beforeTime.getDate()
            const beforeTimeMonth = beforeTime.getMonth() + 1
            const beforeTimeYear = beforeTime.getFullYear()
            
            const listBeforeDay = []
            if (beforeTimeMonth == month) {
                for (let ii = beforeTimeDay; ii < day; ii++) {
                    listBeforeDay.push({
                        day: ii,
                        month,
                        year,
                        active: (this.selectDate.indexOf(`${ii}-${month}-${year}`) >= 0 ) ? 'active' : '',
                         activeMonthClass: this.calClassActiveMonth(month, year) 
                    })
                }
            } else {
                for (let i = beforeTimeDay; i <= numDaysInBeforeMonth; i++) {
                    const itemMonth = this.getBeforeMonthYear(month, year)[0];
                    const itemYear = this.getBeforeMonthYear(month, year)[1];
                    listBeforeDay.push({
                        day: i,
                        month: itemMonth,
                        year: itemYear,
                        active: (this.selectDate.indexOf(`${i}-${itemMonth}-${itemYear}`) >= 0 ) ? 'active' : '',
                        activeMonthClass: this.calClassActiveMonth(itemMonth, itemYear) 
                    })
                }
                for (let ii = 1; ii < day; ii++) {
                    listBeforeDay.push({
                        day: ii,
                        month,
                        year,
                        active: (this.selectDate.indexOf(`${ii}-${month}-${year}`) >= 0 ) ? 'active' : '',
                        activeMonthClass: this.calClassActiveMonth(month, year) 
                    })
                }
            }

            return listBeforeDay;
        },

        getDateMonth (date, month, year, getFull = true) {
            const dateName = this.getNameDate(this.getNumDayOnWeek(date, month, year))
            const monthName = this.getNameMonth(date, month, year);
            return getFull ? `${dateName}, ${monthName} ${date}` : monthName;
        },
        getNumDayOnWeek (date, month, year) {
            var date = new Date()
            date.setDate(date)
            date.setMonth(month - 1)
            date.setFullYear(year)
            return date.getDay() - 1
        },
        getNameDate (dateOnWeek) {
            var d = new Date();
            return d.toString().split(' ')[0];
        },
        getNameMonth (date , month, year) {
            const monthNames = [ "January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
            const d = new Date();
            d.setDate(date)
            d.setMonth(month - 1)
            d.setFullYear(year)
            return monthNames[d.getMonth()]
        },
        getActiveMonth () {
            return this.blurDate.split("-")[0]
        },
        calClassActiveMonth (month, year) {
            const monthBlur = this.blurDate;
            const monthYear = `${month}-${year}`;
            return (monthYear == monthBlur) ? "activeMonth" : "noActiveMonth"
        }, 
    },
    watch: {
        selectDate(data){
            console.log(">>>>>> watch data >>>>>")
            this.$emit('dateSelect', data)
        }
    }

})
</script>
<style scoped>
    .main {
        max-width: 350px;
        height: 350px;
        display: block;
        margin: auto;
        box-sizing: border-box;
    }
    .headInfo {
        width: 100%;
        height: 100px;
        background: #1867c0;
        padding: 10px;
    }
    .headInfoYear {
        color: #9abce3;
        display: block;
    }
    .headInfoMonthDay {
        color: #fff;
        display: block;
        font-size: 30px;
    }
    .calHeader {
        display: block;
        width: 100%;
        height: 50px;
        box-sizing: border-box;
        border-top: 2px solid #4d99b5;
    }
    .itemHeader {
        width: 50px;
        height: 50px;
        display: inline-flex;
        line-height: 50px;
        text-align: center;
        margin: auto;
    }
    .itemHeader span {
        margin: auto;
        text-align: center;
        line-height: 50px;
        width: 100%;
        height: 50px;
        display: block;
        color: #559eb8;
        font-weight: 700;
    }
    .item {
        width: 50px;
        height: 33px;
        float: left;
        /* display: block; */
    }
    .item span:hover {
        background: #f1ecec;
        cursor: pointer;
    }
    .item span {
        margin: auto;
        text-align: center;
        line-height: 30px;
        width: 45px;
        height: 30px;
        display: block;
        font-weight: 600;
        font-size: 13px;
        text-align: end;
        padding: 5px 4px;
        border: 1px solid #d9dcde;
        color: #71aec4;
    }
    .box {
        /* width: 280px; height: 240px; */
        width: 350px;
        height: 198px;
        overflow-y: scroll;
    }

    /* Hide scrollbar for Chrome, Safari and Opera */
    .box::-webkit-scrollbar {
        display: none;
    }
    /* Hide scrollbar for IE, Edge and Firefox */
    .box {
        -ms-overflow-style: none;  /* IE and Edge */
        scrollbar-width: none;  /* Firefox */
    }
    .noActiveMonth {
        color: #c5c9cc !important;
    }
    .active {
        background: #7586bd !important;
        color: #fff !important;
    }
    .controlHeaderText {
        display: block;
        float: left;
        width: 40%;
        height: 50px;
        line-height: 50px;
    }
    .controlHeaderText span {
        font-size: 17px;
        font-weight: 700;
        color: #469fc1;
        padding-left: 2px;
    }
    .controlHeaderBtn {
        display: block;
        float: left;
        width: 60%;
        height: 50px;
        line-height: 50px;
        text-align: right;
        padding-right: 4px;
    }
    .controlHeaderBtn span {
        font-size: 17px;
        font-weight: 700;
        color: #4c7c9f;
        padding-left: 2px;
        text-align: right;
    }
    .controlHeaderBtn span:nth-child(2), .controlHeaderBtn span:nth-child(4) {
        cursor: pointer;
    }
    .btnControl {
        line-height: 50px;
        outline: none;
        border: none;
        background: no-repeat;
        font-size: 18px;
        font-weight: 700;
        color: #4c7c9f;
        cursor: pointer;
    }
</style>





