<template>
    <div class="zss-date-picker" >
        <input class="zss-date-input" :disabled="disabled" type="text" placeholder="请选择日期" readonly v-model="currentValue" @focus="inputFocus" />
        <transition name="fade">
        <div class="zss-picker-model" @click.self="modelClick" v-show="isShow" :style="{'z-index':zIndex}">
            <div class="zss-panel">
                <!-- header -->
                <div class="zss-panel-header">
                    <div class="zss-date-year">
                        <div class="zss-date-prev zss-text-disabled" v-if="select=='year'">＜</div>
                        <div class="zss-date-prev zss-text-hover" v-else @click="prevYear">＜</div>
                        <div class="zss-date-text zss-text-hover" @click="select='year'">{{firstDate[0]}}</div>
                        <div class="zss-date-next zss-text-disabled" v-if="select =='year'">＞</div>
                        <div class="zss-date-next zss-text-hover" v-else @click="nextYear">＞</div>
                    </div>
                    <div class="zss-date-month">

                        <div class="zss-date-prev zss-text-hover" @click="prevMonth">＜</div>
                        <div class="zss-date-text zss-text-hover" @click="select='month'">{{firstDate[1]}}</div>
                        <div class="zss-date-next zss-text-hover" @click="nextMonth">＞</div>
                    </div>
                </div>
                <!-- /header -->
                
                <div class="zss-panel-body">
                    <!-- 年 -->
                    <div class="zss-year-box" v-show="select=='year'">
                        <div class="zss-year" v-for="item in yearList" @click="clickYear(item)">{{item}}</div>
                    </div>
                    <div class="zss-year-control" v-show="select=='year'">
                        <div class="zss-control-prev" @click="prevYears">＜</div>
                        <div class="zss-control-next" @click="nextYears">＞</div>
                    </div>
                    <!-- /年 -->
                    <!--月-->
                    <div class="zss-month-box" v-show="select=='month'">
                        <div class="zss-month" v-for="(item,index) in monthArray" @click="clickMonth(index)">{{item}}</div>
                    </div>
                    <!--/月-->
                    <!-- 周标题 -->
                    <div class="zss-date-week" v-show="select=='date'">
                        <div class="zss-week" v-for="item in weekArray">{{item}}</div>
                    </div>
                    <!-- /周标题 -->
                    <!-- 日期-->
                    <div class="zss-date-box" v-show="select=='date'">
                        <div class="zss-date" :class="{'zss-date-now':isNow(item),'zss-month-now':isMonth(item)}" v-for="item in dateArray" @click="clickDate(item)">{{item[2]}}</div>
                    </div>
                    <!--/日期-->
                </div>                
            </div>
        </div>
        </transition>
    </div>
</template>
<script>
import momnet from 'moment'
import {getMaxZindex} from '../../utils/dom.js'
export default {
    name:'InDatepicker',
    props:{
        value:{
            type:String,
            default:'1990-01-15' //生日 联系我可＋QQ:503305196
        },
        disabled:Boolean
    },
    data(){
        return {
            currentValue:'', //当前值
            zIndex:1,
            isShow:false,
            select:'date',
            weekArray:['日','一','二','三','四','五','六'],
            monthArray:['一月','二月','三月','四月','五月','六月','七月','八月','九月','十月','十一月','十二月'],
            currentDate:[], //当前日期
            dateArray:[],  //当月日期列表
            firstDate:[],  //当月1号
            yearList:[] //当前年列表
        }
    },
    mounted(){
        this.initData();
    },
    methods:{
        //初始化值
        initData(){
            this.currentValue = this.value;
            this.currentDate = (this.value||momnet().format('YYYY-MM-DD')).split('-');
            this.firstDate = momnet(this.currentDate.join('-')).startOf('month').format('YYYY-MM-DD').split('-');
            this.getDateArray();
            this.getYearArray();
            this.getIndex();
        },
        //获取当前页面的最大Index
        getIndex(){
            this.zIndex = getMaxZindex()+1;
        },
        //获取当前显示日期列表
        getDateArray(){
            let day = momnet(this.firstDate.join('-')).day(); //1号是周几
            let list = []
            for(var i = 0; i < 42; i++){
                let date = null;
                if(day == 0){
                    date = momnet(this.firstDate.join('-')).add((i-7),'days');
                }else{
                    date = momnet(this.firstDate.join('-')).add((i-day),'days');
                }
                list.push(date.format('YYYY-MM-DD').split('-'));
            }
            this.dateArray = list;
        },
        //获取当前年列表
        getYearArray(){
             let year = this.firstDate[0];
            let yearList = [];
            for(let i = 0; i < 15; i++){
                yearList.push(year-7+i);
            }
            this.yearList = yearList;
        },
        //获取焦点
        inputFocus(){
           this.initData();
           this.isShow = true;
        },
        //点击遮罩层
        modelClick(){
            this.isShow = false;
        },
        //获取当前日期
        clickDate(item){
            let date = item.join('-');
            this.currentValue = date;
            this.$emit('input',date);
            this.isShow = false;
        },
        //判断是否是当天
        isNow(item){
            if(this.firstDate[1] != this.currentDate[1]){
              return false;
            }
            if(item[0] != this.currentDate[0]){
                return false;
            }
            if(item[1] != this.currentDate[1]){
                return false;
            }
            if(item[2] != this.currentDate[2]){
                return false;
            }
            return true;
        },
        //判断是否当月
        isMonth(item){
            if(item[0] != this.firstDate[0]){
                return false;
            }
            if(item[1] != this.firstDate[1]){
                return false;
            }
            return true;
        },
        //点击前一个月
        prevMonth(){
            let prevMonthDate = momnet(this.firstDate.join('-')).startOf('month').add(-1,'month');
            this.firstDate = prevMonthDate.format('YYYY-MM-DD').split('-');
            this.getDateArray();
        },
        //点击下一个月
        nextMonth(){
            let nextMonthDate = momnet(this.firstDate.join('-')).startOf('month').add(1,'month');
            this.firstDate = nextMonthDate.format('YYYY-MM-DD').split('-');
            this.getDateArray();
        },
        //点击前一年
        prevYear(){
            let prevYearDate = momnet(this.firstDate.join('-')).startOf('month').add(-1,'year');
            this.firstDate = prevYearDate.format('YYYY-MM-DD').split('-');
            this.getDateArray();
        },
        //点击下一年
        nextYear(){
            let nextYearDate = momnet(this.firstDate.join('-')).startOf('month').add(1,'year');
            this.firstDate = nextYearDate.format('YYYY-MM-DD').split('-');
            this.getDateArray();
        },
        //点击上一列表年
        prevYears(){
            let yearList = [];
            for(let i = 0; i < this.yearList.length; i++){
                yearList.push(this.yearList[i]-15);
            }
            this.yearList = yearList;
        },
        //点击下一列表
        nextYears(){
            let yearList = [];
            for(let i = 0; i < this.yearList.length; i++){
                yearList.push(this.yearList[i]+15);
            }
            this.yearList = yearList;
        },
        //点击年
        clickYear(item){
            let firstDate = this.firstDate;
            firstDate.splice(0,1,item+'');
            this.select = 'month';
            this.getDateArray();
        },
        clickMonth(index){
            let month = index+1;
            month = month<10?'0'+month:''+month;
            let firstDate = this.firstDate;
            firstDate.splice(1,1,month);
            this.select = 'date';
            this.getDateArray();
        }
    }
}
</script>