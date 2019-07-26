<!-- 
        手机端时间选择插件
        
        字段
        输入字段                字段名称            类型          默认值
        显示年？                year            Boolean     true
        显示月？                month           Boolean     true
        显示日？                day             Boolean     true
        显示时？                hour            Boolean     false
        显示分？                minites         Boolean     false
        是否显示                show            Boolean     false
        允许滚动时取值     scrollConfirm   Boolean     true
        默认值                 defaultValue    String      new Date().toLocaleDateString()
        
        方法                  值
        changeValue         {year, month, day, hour, minites}
        close               无
-->
<template>
    <div class="date-wrap" v-show="showModal">
        
        <!-- 按钮区域 -->
        <div class="date-btn-list">
            <div class="date-btn-item" @click="cancel">取消</div>
            <div class="date-btn-item" @click="confirm">确认</div>
        </div>

        <!-- 内容区域 -->
        <div class="date-select-wrap">
            <div class="date-select-content">
                <!-- 年份区域 -->
                <div ref="year" class="date-year-select" v-if="showYear">
                    <div>
                        <p></p>
                        <p v-for="item in yearList" :key="item">{{ item }}</p>
                        <p></p>
                    </div>
                    <span>年</span>
                </div>

                <!-- 月份区域 -->
                <div ref="month" class="date-month-select" v-if="showMonth">
                    <div>
                        <p></p>
                        <p v-for="item in monthList" :key="item">{{ item }}</p>
                        <p></p>
                    </div>
                    <span>月</span>
                </div>

                <!-- 天数区域 -->
                <div ref="day" class="date-day-select" v-if="showDay">
                    <div>
                        <p></p>
                        <p v-for="item in dayList" :key="item">{{ item }}</p>
                        <p></p>
                    </div>
                    <span>日</span>
                </div>

                <div ref="hour" class="date-hour-select" v-if="showHour">
                    <div>
                        <p></p>
                        <p v-for="item in hourList" :key="item">{{ item }}</p>
                        <p></p>
                    </div>
                    <span>时</span>
                </div>

                <div ref="minites" class="date-minites-select" v-if="showMinites">
                    <div>
                        <p></p>
                        <p v-for="item in minitesList" :key="item">{{ item }}</p>
                        <p></p>
                    </div>
                    <span>分</span>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    import BScroll from 'better-scroll'

    // 获取年月日时分的初始值
    const min = 2019,
        max = 2050,
        yearList = [],
        monthList = ['01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12'],
        dayList = []

    const days = (year, month) => {
        var  now = new Date(year, month, 0), dayCount = now.getDate()
        return dayCount
    }

    for(let i = min; i <= max; i++){
        yearList.push(i)
    }

    for(let j = 1; j < days(new Date().getFullYear(), new Date().getMonth() + 1); j++){
        dayList.push(('0' + j).slice(-2))
    }

    const getMinites = () => {
        let arr = []
        for(let i = 0; i < 60; i++){
            arr.push(('0' + i).slice(-2))
        }
        return arr
    }


    export default{
        name: 'datePicker',
        props: {
            defaultValue: {     //  默认显示的时间
                type: String,
                required: false,
                default: new Date().toLocaleDateString()
            },
            showYear: {         //  是否显示年份
                type: Boolean,
                required: false,
                default: true
            },
            showMonth: {        //  是否显示月份
                type: Boolean,
                required: false,
                default: true,
            },
            showDay: {          //  是否显示日期
                showDay: Boolean,
                required: false,
                default: true
            },
            showHour: {         //  是否显示小时
                showDay: Boolean,
                required: false,
                default: false
            },
            showMinites: {      //  是否显示分钟
                showDay: Boolean,
                required: false,
                default: false
            },
            show: {             //  是否显示该时间选择组件
                type: Boolean,
                required: false,
                default: false
            },
            scrollConfirm: {    //  是否允许在滚动未停止的时候点击确认按钮
                type: Boolean,
                required: false,
                default: true
            }       
        },
        data() {
            return {
                // 组件的被选择的list数据
                yearList,
                monthList,  
                dayList,
                hourList: [...monthList, '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'],
                minitesList: getMinites(),
                // 选中之后的数据
                year: '',
                month: '',
                day: '',
                hour: '',
                minites: '',
                // 滚动对象
                scrollYear: null,
                scrollMonth: null,
                scrollDay: null,
                scrollHour: null,
                scrollMinites: null,
                // 是否显示该组件
                showModal: true,
            }
        },
        methods: {
            // 初始化滚动效果
            init: function() {
                if(!!this.showYear){        //  优先判断是否需要显示年份
                    if(!this.scrollYear){   //  然后判断年份的scroll组件是否已经初始化
                        this.scrollYear = new BScroll(this.$refs.year,{
                            swipeTime: 1200,
                        })
                        this.scrollYear.on("scrollEnd", pos => {    //  初始化之后绑定滚动终止之后的事件
                            let i = 0
                            while(Math.abs(pos.y + i * 40) > 20){       //  找到scroll位置
                                i++
                            }
                            this.year = this.yearList[i]            //  获取到年份值
                            this.scrollYear.scrollTo(0, -40 * i, 300) //    边缘效果处理
                            this.calculateDay()                     //  年份变化需要重新计算该月的天数
                        })
                    }else{
                        this.scrollYear.refresh()                   //  如果内部的dom节点变化，需要刷新scroll
                    }
                }
                if(!!this.showMonth){
                    if(!this.scrollMonth){
                        this.scrollMonth = new BScroll(this.$refs.month, {
                            swipeTime: 1200,
                        })
                        this.scrollMonth.on("scrollEnd", pos => {
                            let i = 0
                            while(Math.abs(pos.y + i * 40) > 20){
                                i++
                            }
                            this.month = this.monthList[i]
                            this.scrollMonth.scrollTo(0, -40 * i, 300)
                            this.calculateDay()
                        })
                    }else{
                        this.scrollMonth.refresh()
                    }
                }
                if(!!this.showDay){
                    if(!this.scrollDay){
                        this.scrollDay = new BScroll(this.$refs.day, {
                            swipeTime: 1200,
                        })
                        this.scrollDay.on("scrollEnd", pos => {
                            let i = 0
                            while(Math.abs(pos.y + i * 40) > 20){
                                i++
                            }
                            this.day = this.dayList[i]
                            this.scrollDay.scrollTo(0, -40 * i, 300)
                        })
                    }else{
                        this.scrollDay.refresh()
                    }
                }
                if(!!this.showHour){
                    if(!this.scrollHour){
                        this.scrollHour = new BScroll(this.$refs.hour, {
                            swipeTime: 1200,
                        })
                        this.scrollHour.on("scrollEnd", pos => {
                            let i = 0
                            while(Math.abs(pos.y + i * 40) > 20){
                                i++
                            }
                            this.hour = this.hourList[i]
                            this.scrollHour.scrollTo(0, -40 * i, 300)
                        })
                    }else{
                        this.scrollHour.refresh()
                    }
                }
                if(!!this.showMinites){
                    if(!this.scrollMinites){
                        this.scrollMinites = new BScroll(this.$refs.minites, {
                            swipeTime: 1200,
                        })
                        this.scrollMinites.on("scrollEnd", pos => {
                            let i = 0
                            while(Math.abs(pos.y + i * 40) > 20){
                                i++
                            }
                            this.minites = this.minitesList[i]
                            this.scrollMinites.scrollTo(0, -40 * i, 300)
                        })
                    }else{
                        this.scrollMinites.refresh()
                    }
                }   
            },
            // 点击取消
            cancel: function() {
                this.$emit("close")
            },
            // 点击确认
            confirm: function() {
                if(this.scrollConfirm){         //  允许在滚动过程中点击确认按钮选择时间
                    if(!!this.showYear){
                        this.year = this.yearList[Math.round(-this.scrollYear.y / 40)]
                    }
                    if(!!this.showMonth){
                        this.month = this.monthList[Math.round(-this.scrollMonth.y / 40)] 
                    }
                    if(!!this.showDay){
                        this.day = this.dayList[Math.round(-this.scrollDay.y / 40)]
                    }
                    if(!!this.showHour){
                        this.hour = this.hourList[Math.round(-this.scrollHour.y / 40)]
                    }
                    if(!!this.showMinites){
                        this.Minites = this.minitesList[Math.round(-this.scrollMinites.y / 40)]
                    }
                }else{                          //  不允许在滚动过程中确认时间
                    if(!!this.showYear && parseInt(this.scrollYear.y / 40) !== this.scrollYear.y / 40){
                        // console.log("year is scrolling")
                        return
                    }
                    if(!!this.showMonth && parseInt(this.scrollMonth.y/40) !== this.scrollMonth.y / 40){
                        // console.log("month is scrolling")
                        return
                    }
                    if(!!this.showDay && parseInt(this.scrollDay.y / 40) !== this.scrollDay.y / 40){
                        // console.log("day is scrolling")
                        return
                    }
                    if(!!this.showHour && parseInt(this.scrollHour.y / 40) !== this.scrollHour.y / 40){
                        // console.log("hour is scrolling")
                        return
                    }
                    if(!!this.showMinites && parseInt(this.scrollMinites.y / 40) !== this.scrollMinites.y / 40){
                        // console.log("minites is scrolling")
                        return
                    }
                }
                console.log(this.year, this.month, this.day);
                this.$emit("changeValue", {
                    year: this.year,
                    month: this.month,
                    day: this.day,
                    hour: this.hour,
                    minites: this.minites
                })
                this.$emit("close")
            },
            // 当年份或者月份变化的时候，天数页随着重新计算,计算之后刷新scroll滚动
            calculateDay: function() {
                let maxDay = days(this.year, this.month),
                    arr = []
                for(let i = 1; i <= maxDay; i ++){
                    arr.push(('0' + i).slice(-2))
                }
                this.dayList = arr
                this.$nextTick(() => {
                    this.scrollDay.refresh()
                })
            },
            // 根据默认值刷新页面数据，展示父组件传过来的时间
            refreshDataFromDefault: function() {
                let reg = /^(\d{4})[/-](\d{1,2})[/-](\d{1,2})\s?(\d{1,2})?:?(\d{1,2})?$/
                if(reg.test(this.defaultValue)){
                    let y = this.defaultValue.replace(reg, "$1"),
                        m = ('0' + this.defaultValue.replace(reg, "$2")).slice(-2),
                        d = ('0' + this.defaultValue.replace(reg, "$3")).slice(-2),
                        h = ('0' + this.defaultValue.replace(reg, "$4")).slice(-2),
                        n = ('0' + this.defaultValue.replace(reg, "$5")).slice(-2)

                    if(this.showYear){      //  默认值里面的年份初始化
                        for(let i = 0; i < this.yearList.length; i ++){
                            if(y === this.yearList[i]){
                                this.year = y
                                this.scrollYear.scrollTo(0, -40 * i, 500)
                                break
                            }
                        }
                    }
                    if(this.showMonth){     //  默认值里面的月份初始化
                        for(let i = 0; i < this.monthList.length; i ++){
                            if(m === this.monthList[i]){
                                this.month = m
                                this.scrollMonth.scrollTo(0, -40 * i, 500)
                                break
                            }
                        }
                    }
                    if(this.showDay){       //  默认值里面的日期初始化
                        for(let i = 0; i < this.dayList.length; i ++){
                            if(d === this.dayList[i]){
                                this.day = d
                                this.scrollDay.scrollTo(0, -40 * i, 500)
                                break
                            }
                        }
                    }
                    if(this.showHour){      //  默认值里面的小时初始化
                        for(let i = 0; i < this.hourList.length; i ++){
                            if(h === this.hourList[i]){
                                this.hour = h
                                this.scrollHour.scrollTo(0, -40 * i, 500)
                                break
                            }
                        }
                    }
                    if(this.showMinites){   //  默认值里面的分钟初始化
                        for(let i = 0; i < this.minitesList.length; i ++){
                            if(n === this.minitesList[i]){
                                this.minites = n
                                this.scrollMinites.scrollTo(0, -40 * i, 500)
                                break
                            }
                        }
                    }
                }else{
                    console.log("参数格式不正确,请确认时间格式之后再测试！")
                }
            }
        },
        watch: {
            // 父组件控制时间选择组件是否显示
            show: function() {
                this.showModal = this.show
                if(!!this.show){
                    this.$nextTick(() => {
                        this.init()
                        if(!!this.defaultValue){        //  优先显示默认传过来的时间
                            this.refreshDataFromDefault()
                        }
                    })
                }
            },
        },
        mounted: function() {
            this.init()
        }
    }
</script>
<style scoped>
    .date-wrap{
        position: fixed;
        bottom: 0;
        left: 0;
        z-index: 99;
        width: 100%;
        height: 240px;
        font-size: 13px;
        background: #ffffff;        
    }
    .date-btn-list{
        overflow: hidden;
        color: green;
        background: #f2f2f2;
    }
    .date-btn-item{
        display: inline-block;
        padding: 5px 10px;
        height: 30px;
        line-height: 30px;
    }
    .date-btn-item:nth-child(1){
        float: left;
        color: #888888;
    }
    .date-btn-item:nth-child(2){
        float: right;
    }
    .date-select-wrap{
        position: relative;
        width: 100%;
    }
    .date-select-content{
        display: flex;
        position: relative;
        margin-top: 40px;
        width: 100%;
        height: 120px;
    }
    .date-select-content::before{
        content: "";
        position: absolute;
        left: 0;
        top: 40px;
        width: 100%;
        height: 1px;
        background-color: #ccc;
        transform: scaleY(.5);
        -webkit-transform: scaleY(.5);
    }
    .date-select-content::after{
        content: "";
        position: absolute;
        left: 0;
        top: 80px;
        width: 100%;
        height: 1px;
        background-color: #ccc;
        transform: scaleY(.5);
        -webkit-transform: scaleY(.5);
    }
    .date-select-content>div{
        position: relative;
        flex: 1;
        overflow: hidden;
        height: 120px;
        text-align: center;
    }
    .date-select-content>div::before{
        content: '';
        position: absolute;
        top: 0;
        left: 0;
        z-index: 999;
        width: 100%;
        height: 40px;
        background: linear-gradient(180deg, rgba(255, 255, 255, 1), rgba(255, 255, 255, .7), rgba(255, 255, 255, .1));
    }
    .date-select-content>div::after{
        content: '';
        position: absolute;
        top: 80px;
        left: 0;
        z-index: 999;
        width: 100%;
        height: 40px;
        background: linear-gradient(0deg, rgba(255, 255, 255, 1), rgba(255, 255, 255, .7), rgba(255, 255, 255, .1));
    }
    .date-select-content>div div>p{
        height: 40px;
        line-height: 40px;
        margin: 0;
        padding: 0;
    }
    .date-select-content>div div>span{
        position: absolute;
        top: 42%;
        left: 65%;
    }
    .date-select-content>div div.date-year-select{
        &>span{
            left: 80%;
        }
    }
</style>