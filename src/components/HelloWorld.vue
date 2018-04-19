<template slot-scope="scope">
  <div class="hello">
  	<div style="margin: 20px 0; display: flex">
      <div  style="display: flex">
        <div style="height: 40px; line-height:40px;padding: 0 20px 0 0;"><b>厂商名称</b></div>
         <el-select v-model="value2" filterable placeholder="请选择" @change="select_one">
          <el-option
            v-for="item in results"
            :key="item.union_id"
            :label="item.union_name"
            :value="item.union_id"
          >
          </el-option>
        </el-select>
      </div>

     <div style="display: flex;margin-left: 50px;">
       <div style="height: 40px; line-height:40px;padding: 0 20px 0 0;"><b>车场名称</b></div>
          <el-select v-model="value3" filterable placeholder="请选择" @change="select_two">
            <el-option
              v-for="item in results_two"
              :key="item.park_id"
              :label="item.park_name"
              :value="item.park_id"
              >
            </el-option>
         </el-select>
      </div>
    </div>
			  <el-table
			    :data="results_table"
			    border
			    style="text-aligin:center;margin: 0 auto;">
			    <el-table-column
			      prop="union_id"
			      label="厂商编号"
			      width="180">
			    </el-table-column>
			    <el-table-column
			      prop="park_id"
			      label="车场编号"
			      width="180">
			    </el-table-column>
			    <el-table-column
			      prop="park_name"
			      label="车场名称">
			    </el-table-column>
          <el-table-column
            prop="local_id"
            label="local_id">
          </el-table-column>
          <el-table-column
            prop="lastbeat_date"
            label="心跳时间">
          </el-table-column>
          <el-table-column
            prop=""
            label="选择日期">
            <template slot-scope="scope">
              <el-button
                size="mini"
                native-type="button"

                @click="view_log(scope.$index, scope.row)">查看日志</el-button>
            </template>
           </el-table-column>
            <el-table-column
              prop="local_id"
              label="下载">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  :disabled="isDisabled"
                  native-type="button"
                  @click="Downloading_log(scope.$index, scope.row)">下载</el-button>
              </template>
            </el-table-column>
          </el-table>
          <el-dialog
            title="提示"
            :visible.sync="centerDialogVisible"
            width="30%"
            center>
            <div class="block" style="display: flex">
              <span class="demonstration" style="heightL:40px;line-height: 40px;padding: 0 20px 0 0;"><b>日志日期</b></span>
              <el-date-picker
                v-model="time"
                type="date"
                value-format="yyyyMMdd"
                placeholder="选择日期" >
              </el-date-picker>
            </div>
            <span slot="footer" class="dialog-footer">
              <el-button native-type="button" style="margin-left: 75px;" @click="centerDialogVisible = false">取 消</el-button>
              <el-button native-type="button" style="background: darkgreen;color: white;"  @click="determine()">确 定</el-button>
            </span>
          </el-dialog>
     </div>
</template>

<script>
export default {
  name: 'er_jld',
  data() {
     return {
       //日期控件
       pickerOptions1: {
         disabledDate(time) {
           return time.gettime() > Date.now();
         }
       },
         results:[],
         results_two:[],
         results_table:[],
         value2: '',
         value3:'',
         value1: '',
         time: '',
         centerDialogVisible: false,
         isDisabled:true,
         res_data:[]

      }
    },
      mounted(){
        this.$ajax.get("http://test.bolink.club/logview/getunion")
          .then(response => {
            this.results = response.data;
          })
        },
      methods:{
//        发送setIntval请求
        setTime_out(){
          var _this = this;
          var row = this.row;
           setInterval(function(){
             _this.$ajax.get("http://test.bolink.club/logview/log/fileisexist?union_id="+row.union_id+"&park_id="+row.park_id+"&local_id="+row.local_id+"&log_time="+row.log_time)
               .then(response => {
                 _this.res_data = response.data;
//                 console.log(_this.res_data,'++++++++++++++++')
                 if(_this.res_data.state == 0){
                   _this.isDisabled = true;
                 }else{
                   _this.isDisabled = false;
                 }
               })
           },3000)
        },
        //二级联动父级
         select_one(val){
           this.$ajax.get("http://test.bolink.club/logview/getpark?union_id="+val)
             .then(response => {
               this.results_two = response.data;
             })
           },
        //二级联动子级
         select_two(val){
          this.$ajax.get("http://test.bolink.club/logview/getparkinfo?park_id="+val)
            .then(response => {
              this.results_table = response.data;
            })
          },
        //查看日志
         view_log(index,row){
             this.centerDialogVisible = true;
             this.row = row;
              this.$ajax.get("http://test.bolink.club/logview/log/noticesdkuploadlog?union_id="+row.union_id+"&park_id="+row.park_id+"&local_id="+row.local_id+"&log_time="+row.log_time)
                .then(response => {
                  this.results_two = response.data;
                })
           },
        //下载
         Downloading_log(index,row){
            location.href = "http://test.bolink.club/logview/log/download?union_id="+row.union_id+"&park_id="+row.park_id+"&local_id="+row.local_id+"&log_time="+row.log_time;
          },


        //点击确定发送table>tr数据
         determine(){
//            this.isDisabled =false;
            var row = this.row;
            this.$ajax.get("http://test.bolink.club/logview/log/noticesdkuploadlog?union_id="+row.union_id+"&park_id="+row.park_id+"&local_id="+row.local_id+"&log_time="+row.log_time)
              .then(response => {
                this.results_two = response.data;
                console.log(this.results_two)
                console.log(this.time)
              })
               this.centerDialogVisible=false
               this.setTime_out()
           },
         }
      }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
