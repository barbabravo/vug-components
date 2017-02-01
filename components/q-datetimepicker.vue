<template>
<div class="q-datetimepicker" :data-cache-id="cache_id">
    <input class="q-input" type="text" :value="model" @click="toggleBox($event)" :placeholder="placeholder" :disabled="disabled" readonly="readonly"><span class="q-icon glyphicon glyphicon-calendar"></span>
    <div class="q-box" v-show="show_box==true">
        <div class="q-box-detail">
            <table class="q-box-calendar">
                <thead>
                    <tr class="q-box-title">
                        <td colspan="1" class="left"><i class="jumper" @click="prev"><</i></td>
                        <td colspan="5" class="center"><span v-text="date|formatDate 'yyyy-MM'"></span></td>
                        <td colspan="1" class="right"><i class="jumper" @click="next">></i></td>
                    </tr>
                </thead>
                <tbody>
                    <tr class="q-box-label">
                        <td v-for="(index, day) in week"><strong v-text="day" :class="{'text-danger':index==0||index==week.length-1}"></strong></td> 
                    </tr>
                    <tr class="q-box-week" v-for="(week_index, week) in cur_month_days" track-by="$index">
                        <td class="q-box-day" 
                            v-for="(day_index, day) in week" 
                            @click="selectDay(day)"
                            :class="{
                                'current-day': day.current_day,
                                'before-current-day': day.before_current_day,
                                'before-current-month': day.before_current_month,
                                'after-current-month': day.after_current_month,
                                'disabled':day.disabled}">
                            <a class="day-text" v-text="day.timestamp | formatDate format"></a>
                        </td> 
                    </tr>
                </tbody>
            </table>
            <!-- <div class="q-box-time">
                <input type="text" value="10:33:22">
            </div> -->
        </div>
    </div>
</div>
</template>


<script>

export default {
    props:{
        "model":{
            type:String,
            default:(new Date().getFullYear() + '-' + (new Date().getMonth() + 1) + '-' + new Date().getDate())
        },
        "format":{
            type:String,
            default:'MM-dd'
        },
        "min_date":{
            type:String
        },
        "max_date":{
            type:String
        }
    },
    data(){
        return {
            show_box:false,
            cache_id:new Date().getTime() + '-' + parseInt(Math.random()*1000),
            week:['周日','周一','周二','周三','周四','周五','周六'],
            date:null
        }
    },
    created(){
        var self = this;
        var monitor = function(e){
            var root_node = document.querySelector(".q-datetimepicker[data-cache-id='" + self.cache_id + "']");
            // if(root_node && e.target.parentNode != root_node.querySelector('.q-box-calendar')){
            //     self.show_box = false;
            // }
        }

        if(document.addEventListener){
            document.addEventListener('click', monitor, true);
        }else{
            document.attachEvent('onClick', monitor, true);
        }
    },
    computed:{
        model_date(){
            return new Date(this.model);
        },
        cur_year(){
            return this.date.getFullYear();
        },
        cur_month(){
            return this.date.getMonth() + 1;
        },
        cur_day(){
            return this.date.getDate();
        },
        cur_week_day(){
            return this.date.getDay();
        },
        cur_month_days(){
            var decorated_list = [];
            var list = [];

            var first_weekdate = new Date(this.cur_year, this.cur_month-1, 1);

            var first_weekday = first_weekdate.getDay();

            var total_days = this.getDaysInMonth(this.cur_year, this.cur_month, 0);
            //能铺满整个表格
            if(total_days%7>0){
                total_days = total_days + 7 - total_days%7;
            }

            for(var i = 0; i< total_days; i++){
                var date = this.getDateByOffsetDay(first_weekdate, i - first_weekday);

                var date_item = {
                    date:date,
                    timestamp:date.getTime()
                };

                if(date.getFullYear() == this.date.getFullYear()){
                //当前年
                    if(date.getMonth() == this.date.getMonth()){
                    //当前月
                        date_item.current_month = true;
                        if(date.getDate() == this.date.getDate()){
                            //当天
                            if(date.getDate() == this.model_date.getDate()){
                                date_item.current_day = true;
                            }
                        }else if(date.getDate() > this.date.getDate()){
                        //当月之后的天
                            date_item.after_current_day = true;
                        }else{
                        //当月之前的天
                            date_item.before_current_day = true;
                        }
                    }else if(date.getMonth() > this.date.getMonth()){
                    //之后的月
                        date_item.after_current_month = true;
                    }else{
                    //之前的月
                        date_item.before_current_month = true;
                    }
                }else if(date.getFullYear() > this.date.getFullYear()){
                    date_item.after_current_month = true;
                }else{
                    date_item.before_current_month = true;
                }

                if(this.min_date){
                    var min_date = new Date(this.min_date)
                    if(date.getTime() < min_date.getTime()){
                        date_item.before_min_date = true;
                    }
                }
                if(this.max_date){
                    var max_date = new Date(this.max_date)
                    if(date.getTime() > max_date.getTime()){
                        date_item.after_max_date = true;
                    }
                }

                if(date_item.before_min_date || date_item.after_max_date){
                        date_item.disabled = true;
                }


                list.push(date_item);

                if(i>0 && (i+1)%7==0){
                    decorated_list.push(list.slice(0));
                    list = [];
                }
            }
            return decorated_list;
        }
    },
    methods:{
        toggleBox: function(e){
            this.show_box = !this.show_box;
            if(this.show_box){
                if(this.modal){
                    this.date = new Date(this.model);
                }else{
                    this.date = new Date();
                }
                if(!this.date){
                    console.error("错误的日期格式!")
                }
            }
        },
        getDaysInMonth: function(year, month){
            return new Date(year, month, 0).getDate();
        },
        getDateByOffsetDay: function(date, offset){
            return new Date(date.getTime() + offset*24*60*60*1000);
        },
        jumpToDate: function(date){
            this.date = date;
        },
        prev: function(){
            var date = this.getDateByOffsetDay(new Date(this.cur_year, this.cur_month-1, 1), -1);
            this.jumpToDate(date)
        },
        next: function(){
            var date = this.getDateByOffsetDay(new Date(this.cur_year, this.cur_month, 0), 1);
            this.jumpToDate(date)
        },
        formatDate: function(timestamp, format){
            var timestamp = timestamp || 0, default_value = default_value || '', ret;

            var covertNumberToTwoDigit = function(number) {
              return number.toString().replace(/^(\d)$/, '0$1');
            }
            if(!timestamp || timestamp == '0') {
              ret = default_value;
            }else{
              if(timestamp.toString().length == 10){
                timestamp = timestamp * 1000;
              }
              var date = new Date(timestamp),
              full_year = date.getFullYear() || '0000',
              month = covertNumberToTwoDigit(date.getMonth() + 1) || '00',
              day = covertNumberToTwoDigit(date.getDate()) || '00',
              hour = covertNumberToTwoDigit(date.getHours()) || '00',
              minute = covertNumberToTwoDigit(date.getMinutes()) || '00',
              second = covertNumberToTwoDigit(date.getSeconds()) || '00';
              ret = format.replace(/yyyy/, full_year).replace(/MM/, month).replace(
                /dd/, day).replace(/HH/, hour).replace(/mm/, minute).replace(
                  /ss/,
                  second);
            }
            return ret;
        },
        selectDay: function(day){
            if(day.disabled) return;
            var select_day = this.formatDate(day.timestamp, 'yyyy-MM-dd')
            this.model = select_day;
            this.show_box = false;
            this.$dispatch('on-select', select_day);
        }
    }
}
</script>


<style lang="less" scoped>
.q-datetimepicker{
    position:relative;
    .q-input{
        box-sizing:border-box;
        -webkit-box-sizing:border-box;
        -moz-box-sizing:border-box;
        width:100%;
        height:100%;
        border:none;
    }
    .q-icon{
        display:block;
        position: absolute;
        right: 10px;
        top: 7px;
        color: #ccc;
        font-size: 20px;
    }

    .q-box{
        position:absolute;
        z-index:9999;
        left:0;
        &.hidden{
            display:none;
        }
        .q-box-detail{
            margin-top:10px;
            border:1px solid #ccc;
            background-color:#FFF;
            box-shadow:0 0 8px rgba(0,0,0,.2);
            padding:5px;
            .q-box-time,
            .q-box-calendar{
                min-width:240px;
                width:100%;
            }
            .q-box-time{
                input{
                    width:100%;
                    border:0 none;
                }
            }
            .q-box-calendar{
                thead{
                    .left{
                        width:40px;
                    }
                    .right{
                        width:40px;
                    }
                    .center{
                        width:160px;
                    }
                }
                tbody{
                    border-top:1px solid #ccc;
                }
                tr{
                }
                td{
                    text-align:center;
                    strong{
                        font-size:120%;
                        padding:10px;
                        display:block;

                    }
                    .jumper,
                    a{
                        display:block;
                        text-decoration: none;
                        color:#000;
                        padding:10px;
                        width:100%;
                        height:100%;
                        white-space: nowrap;
                        cursor:pointer;
                        &:hover{
                            color:#FFF;
                            background-color:#5cb85c;
                        }                     
                    }
                    &.before-current-month,
                    &.after-current-month{
                        a,a:link{
                            color:#ccc;
                        }
                    } 
                    &.current-day{
                        a,a:link{
                            color:#FFF;
                            background-color:#5cb85c;
                        }
                    }
                    &.disabled{
                        a{
                            cursor:not-allowed;
                            color:#ccc;
                        }
                    }
                }
            }
        }
    }
    .q-box-input{
        display:block;
        width:100%;
        height:100%;
    }
}
</style>

