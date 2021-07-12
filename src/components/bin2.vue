<template>
    <h2 style="text-align: center;"> Date Custom</h2>
    <span> List seleect date: </span>
    
    <span style="" v-for="item in selectDate" :key="item" >
         {{ item }} / 
    </span>

    <div class="main">
        <!-- <div class="headInfo">
            <span class="headInfoYear"> {{ yearNow }} - {{ dateMonth }} </span>
            <span class="headInfoMonthDay"> {{ blurDate }} </span>
        </div>
        <br /> -->

        <div class="controlHeader">
            <div class="controlHeaderText">
                <span> Điểm gửi </span>
            </div>
            <div class="controlHeaderBtn">
                <span> May </span>
                <span> &lt;	 </span>
                <span> {{ blurDate }} </span>
                <span> &gt; </span>
            </div>
        </div>
        <div style="clear: both"></div>
        <div class="calHeader">
            <div class="itemHeader"> <span> Sun </span> </div>
            <div class="itemHeader"> <span> Mon </span> </div>
            <div class="itemHeader"> <span> Tue </span> </div>
            <div class="itemHeader"> <span> Wed </span> </div>
            <div class="itemHeader"> <span> Thu </span> </div>
            <div class="itemHeader"> <span> Fri </span> </div>
            <div class="itemHeader"> <span> Sat </span> </div>
        </div>

        <div id="boxId" class="box" v-on:scroll="handleScroll" ref="boxRef"> 
            <div 
                v-for="item in listDate" :key="item.day" 
                :id="item.day + '_' + item.month + '_' + item.year"
                :class="'item ' + item.activeMonthClass"
                @mouseover="setBlurDate(item.day, item.month, item.year)"
            >
                <span 
                :class="{'active': item.active }"
                @click="clickDateAction(item.day, item.month, item.year)">
                    {{ item.day }}
                </span>
            </div>
        </div>

        <br /><br /><br />
    </div>
</template>

<script>
import { defineComponent, ref } from 'vue'

export default defineComponent({
    name: 'DateCustom',
    props: ["_date", "_month", "_year", "_activeDateList"],
    data() {
        return {
            year: "",
            month: "",
            date: "",
            listDate: [], // List Date For Render HTML
            selectDate: [], // List Date Selected 

            currentMonth: "",
            currentYear: "",
            beforeDateState: "",
            nextDateState: "",

            dateNow: "",
            monthNow: "",
            yearNow: "",
            dateMonth: "",
            blurDate: ""
        }
    },
    created () {
        window.addEventListener('scroll', this.handleScroll);
    },
    unmounted () {
        window.removeEventListener('scroll', this.handleScroll);
    },

    mounted() {
        console.log(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> Mounted")
        if (this._activeDateList && this._activeDateList.length > 0) this.selectDate = [...this.selectDate, ...this._activeDateList]
        
        const now = new Date()
        this.year = now.getFullYear()
        this.month = now.getMonth() + 1
        this.date = now.getDate()

        var date = this.date
        var month = this.month
        var year = this.year

        this.dateNow = date
        this.monthNow = month
        this.yearNow = year
        this.dateMonth = this.getDateMonth(date, month, year)

        if (this._date) date = Number(this._date)
        if (this._month) month = Number(this._month)
        if (this._year) year = Number(this._year)

        this.blurDate = `${month}-${year}`
        
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

        const offDayHead = firstDayOnMonth.getDay()

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
                active: (this.selectDate.indexOf(`${ii}-${month}-${year}`) >= 0 ) ? true : false,
                activeMonthClass: this.calClassActiveMonth(month, year)
            })
        }

        // Cal List Day Head
        const [monthHeadListBefore, yearHeadListBefore] = this.getBeforeMonthYear(month, year)
        timeOffHeadList = this.getListDayEndOfMonth(offDayHead, monthHeadListBefore, yearHeadListBefore )
        // Set beforeDateState
        this.beforeDateState = timeOffHeadList[0]

        // Cal List Day End
        // const [monthNextListBefore, yearNextListBefore] = getNextMonthYear(month, year)

        const numCellEndList = 42 - Number(timeMainList.length) - Number(timeOffHeadList.length) + 14 // 14 is 2 line add before for top scroll
        timeNextList = this.getListDayStartOfMonth(numCellEndList, month, year)

        // Set State nextDateState
        this.nextDateState = timeNextList.slice(-1)[0] 
        const fullDayListBox = [...timeOffHeadList , ...timeMainList, ...timeNextList]

        // 42 day
        console.log({
            timeOffHeadList,
            timeMainList,
            timeNextList,
            numCellEndList,
            fullDayListBox,
            nextDateState: this.nextDateState,
            beforeDateState: this.beforeDateState
        })

        this.listDate = fullDayListBox;
        const time = []; // Array bin 
        for (let i = 1; i <= this.listDate.length; i++) {
            const itemKeyBind = `${fullDayListBox[i-1]["day"]}-${fullDayListBox[i-1]["month"]}-${fullDayListBox[i-1]["year"]}`
            const isActiveClass = this.selectDate.indexOf(itemKeyBind);
            time.push({
                day: fullDayListBox[i-1]["day"],
                month: fullDayListBox[i-1]["month"],
                year: fullDayListBox[i-1]["year"],
                active: (isActiveClass > -1) ? true : false,
                activeMonthClass: this.calClassActiveMonth(fullDayListBox[i-1]["month"], fullDayListBox[i-1]["year"])
            })
        }
        this.listDate = time;

        console.log("==========================================================================================")
        setTimeout(function() {
            var div = document.getElementById("boxId");
            div.scrollTop = 70 
        }, 10)
    },
    methods: {
        setBlurDate (date, month, year) {
            this.blurDate = `${month}-${year}`
        },
        handleScroll ({ target: { scrollTop, clientHeight, scrollHeight }}) {
            const { day, month, year } = this.nextDateState;
            const dayB = this.beforeDateState.day
            const monthB = this.beforeDateState.month
            const yearB = this.beforeDateState.year

            if (scrollTop + clientHeight >= scrollHeight) {
                console.log(">>>>>>>> Buttonnn")
                const nextDayList = this.getNextDayWithAnyDay(day, month, year);
                this.listDate = [...this.listDate, ...nextDayList]
                this.nextDateState = nextDayList.slice(-1)[0]
                console.log(">>>>>>>> this.nextDateState ::", nextDayList.slice(-1)[0])
            }

            if (scrollTop == 0) {
                console.log(">>>>>>>> Topppppp")
                const beforeDayList = this.getBeforeDayWithAnyDay(dayB, monthB, yearB)
                this.listDate = [ ...beforeDayList, ...this.listDate]

                this.beforeDateState = beforeDayList[0]

                console.log(">>>>>>>> day, month, year:", dayB, monthB, yearB)
                console.log(">>>>>>>>>>>>> beforeDayList::", beforeDayList)
                console.log(">>>>>>>>>>>>> beforeDateState::", beforeDayList[0])

                var div = document.getElementById("boxId");
                div.scrollTop = 1;
            }
        },
    
        clickDateAction (day, month, year) {
            const selectDateFormat = `${day}-${month}-${year}`;

            // console.log(">>>>>>>>> XXXX day:", selectDateFormat)
            // console.log(">>>>>>>>> selectDate:", this.selectDate)
            // console.log(">>>>>>>>> XXXX listDate:", this.listDate)

            if (!this.selectDate) this.selectDate = []
            const countDateExit = this.selectDate.indexOf(selectDateFormat);

            this.listDate.map(item => {
                if (item.day == day && item.month == month && item.year == year) {
                    if (item.active) {
                        item.active = false
                    } else {
                        item.active = true
                    }
                }
            })

            if (countDateExit == -1) {
                 this.selectDate = [...new Set(this.selectDate), selectDateFormat]
            } else {
                const a = this.selectDate
                const r = []
                a.map((item) => {
                    if (item != selectDateFormat) {
                        r.push(item)
                    }
                })
                this.selectDate = r
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
            if (month == 1) return [12, year - 1]
            return [(month -1), year]
        },
        // Get Next Month Year
        getNextMonthYear (month, year) {
            if (month == 12) return [1, year + 1]
            return [(month +1), year]
        },

        // Get List Day End Of Month From Num 
        getListDayEndOfMonth (num, month, year) {
            var list = []
            var numDaysInMonth = new Date(year, month, 0).getDate(); // numDaysInMonth(month, year)
            var daySatrtCal = numDaysInMonth - num + 1 - 14 // xxxx //

            for (let i = daySatrtCal; i <= numDaysInMonth; i++) {
                const itemMonth = this.getBeforeMonthYear((month + 1), year)[0];
                const itemYear = this.getBeforeMonthYear((month + 1), year)[1];
                list.push({
                    day: i,
                    month: itemMonth,
                    year: itemYear,
                    active: (this.selectDate.indexOf(`${i}-${itemMonth}-${itemYear}`) >= 0 ) ? true : false ,
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
                    active: (this.selectDate.indexOf(`${i}-${itemMonth}-${itemYear}`) >= 0 ) ? true : false,
                     activeMonthClass: this.calClassActiveMonth(itemMonth, itemYear)
                })
            }
            return list
        },

        getNextDayWithAnyDay (day , month, year) {
            let numDayNext = 14; // This is config default

            var timeInit = new Date()
            timeInit.setDate(day)
            timeInit.setMonth(month - 1)
            timeInit.setFullYear(year)

            const numDaysInMonth = this.numDaysInMonth(month, year)
            const [nextMonth, nextYear] = this.getNextMonthYear(month, year)

            let nextTime = timeInit
            nextTime.setDate(nextTime.getDate() + numDayNext);

            const nextTimeDay = nextTime.getDate()
            const nextTimeMonth = nextTime.getMonth() + 1
            const nextTimeYear = nextTime.getFullYear()
            
            const listNextDay = []
            if (nextTimeMonth == month && day != numDaysInMonth) {
                for (let ii = (day + 1); ii <= (day + numDayNext); ii++) {
                    listNextDay.push({
                        day: ii,
                        month,
                        year,
                        active: (this.selectDate.indexOf(`${ii}-${month}-${year}`) >= 0 ) ? true : false,
                         activeMonthClass: this.calClassActiveMonth(month, year)
                    })
                }
            } else {
                for (let i = (day + 1); i <= numDaysInMonth; i++) {
                    listNextDay.push({
                        day: i,
                        month,
                        year,
                        active: (this.selectDate.indexOf(`${i}-${month}-${year}`) >= 0 ) ? true : false,
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
                        active: (this.selectDate.indexOf(`${ii}-${itemMonth}-${itemYear}`) >= 0 ) ? true : false,
                         activeMonthClass: this.calClassActiveMonth(itemMonth, itemYear)
                    })
                }
            }

            return listNextDay;
        },
        
        getBeforeDayWithAnyDay (day, month, year) {
            let numDayBefore = 14; // This is config default

            var timeInit = new Date()
            timeInit.setDate(day)
            timeInit.setMonth(month - 1)
            timeInit.setFullYear(year)

            const numDaysInMonth = this.numDaysInMonth(month, year)
            const numDaysInBeforeMonth = this.numDaysInMonth(this.getBeforeMonthYear(month, year)[0], year)

            const [beforeMonth, beforeYear] = this.getBeforeMonthYear(month, year)

            let beforeTime = timeInit
            beforeTime.setDate(beforeTime.getDate() - numDayBefore);

            const beforeTimeDay = beforeTime.getDate()
            const beforeTimeMonth = beforeTime.getMonth() + 1
            const beforeTimeYear = beforeTime.getFullYear()

            // console.log(">>>>>> LOg Iniot Test Beforeee::", {
            //     numDaysInMonth,
            //     numDaysInBeforeMonth,
            //     beforeMonth,
            //     beforeYear,
            //     beforeTimeDay,
            //     beforeTimeMonth,
            //     beforeTimeYear
            // });
            
            const listBeforeDay = []
            if (beforeTimeMonth == month) { // && day != numDaysInMonth
                for (let ii = beforeTimeDay; ii < day; ii++) {
                    listBeforeDay.push({
                        day: ii,
                        month,
                        year,
                        active: (this.selectDate.indexOf(`${ii}-${month}-${year}`) >= 0 ) ? true : false,
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
                        active: (this.selectDate.indexOf(`${i}-${itemMonth}-${itemYear}`) >= 0 ) ? true : false,
                         activeMonthClass: this.calClassActiveMonth(itemMonth, itemYear) 
                    })
                }
                for (let ii = 1; ii < day; ii++) {
                    listBeforeDay.push({
                        day: ii,
                        month,
                        year,
                        active: (this.selectDate.indexOf(`${ii}-${month}-${year}`) >= 0 ) ? true : false,
                        activeMonthClass: this.calClassActiveMonth(month, year) 
                    })
                }
            }

            return listBeforeDay;
        },

        getDateMonth (date, month, year) {
            const dateName = this.getNameDate(this.getNumDayOnWeek(date, month, year))
            const monthName = this.getNameMonth()
            return `${dateName}, ${monthName} ${date}`
        },
        getNumDayOnWeek (date, month, year) {
            var _date = new Date()
            _date.setDate(date)
            _date.setMonth(month - 1)
            _date.setFullYear(year)
            return _date.getDay()
        },
        getNameDate (dateOnWeek) {
            var d = new Date();
            return d.toString().split(' ')[0];
        },
        getNameMonth () {
            const monthNames = [ "January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
            const d = new Date();
            return monthNames[d.getMonth()]
        },
        getActiveMonth () {
            return this.blurDate.split("-")[0]
        },
        calClassActiveMonth (month, year) {
            const monthBlur = this.blurDate;
            const monthYear = `${month}-${year}`;
            return (monthYear == monthBlur) ? "activeMonth" : ""
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
        font-weight: 500;
        font-size: 13px;
        text-align: end;
        padding: 5px 4px;
        border: 1px solid #d9dcde;
        color: #71aec4;
    }

    .box {
        /* width: 280px;
        height: 240px; */
        width: 350px;
        height: 210px;
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

    

    .active {
        background: #7586bd !important;
        color: #fff !important;
    }

    .activeMonth {
        /* background: #eae5e5; */
        /* border: 1px solid red; */
    }



    /*  */
    .controlHeader {
        /* border-top: 2px solid #4d99b5; */
    }

    .controlHeaderText {
        display: block;
        float: left;
        width: 40%;
        height: 50px;
        line-height: 50px;
        /* background: yellow; */
    }
    .controlHeaderText span {
        font-size: 17px;
        font-weight: 700;
        color: #4c7c9f;
        padding-left: 2px;
    }

    .controlHeaderBtn {
        display: block;
        float: left;
        width: 60%;
        height: 50px;
        line-height: 50px;
        /* background: pink; */
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

    
</style>





