<template>
  <div id = "uploadContainer">
    <el-popover
        placement="bottom-start"
        title="上传总进度"
        width="700"
        trigger="hover"
        id = "uploadPopover">
      <el-button size="small" type="primary" class = "popButton" @click = "routeToUpload" round>前往上传主页面</el-button>
      <div class= "crossLine"></div>
      <el-progress id = "uploadTotalBar" :text-inside="true" :stroke-width="20" :percentage="totalProgress"  status="success"></el-progress>
      <div class="container">
        <el-table
            :data="uploadingData"
            class = "tmpTable"
            height="400"
            empty-text = "暂无正在上传的文件"
            :header-cell-style="center"
            :row-style="{background:'transparent',color:'white'}"
        >
          <el-table-column
              label="文件名"
              prop= "name">
            <template v-slot:default="scope">
              <i :class="'bi-'+ scope.row.fileType"></i>
              <span style="margin-left: 10px">{{ scope.row.name }}</span>
            </template>
          </el-table-column>
          <el-table-column
              label="上传速度"
              prop="speed"
              width="95">
          </el-table-column>
          <el-table-column
              label="剩余时间"
              prop="leftTime"
              width="95">
          </el-table-column>
          <el-table-column
              label="状态"
              prop="status"
              width="95">
          </el-table-column>
          <el-table-column
              label="上传进度"
              prop="complete">
            <template v-slot:default="scope">
              <el-progress :text-inside="true" :stroke-width="24" :percentage="scope.row.complete" status="success"></el-progress>
            </template>
          </el-table-column>

          <el-table-column
              width="100"
              align="right">
          <template v-slot:default="scope">
            <el-button v-if="!scope.row.isPause && scope.row.status == '正在上传'" size="mini" type="warning" icon="bi-pause" @click = "paused(scope.row.id)" circle></el-button>
            <el-button v-else-if="scope.row.isPause && scope.row.status == '已暂停'" size="mini" type="success" icon="bi-play" @click = "unPaused(scope.row.id)" circle></el-button>
            <el-button v-if="scope.row.status !== '准备中..' && scope.row.status !== '暂停中..' && scope.row.status !== '中止中..' && scope.row.status !== '少女祈祷中..'" size="mini" type="danger" icon="el-icon-close" @click = "abortMission(scope.row.id,scope.row.status)" circle></el-button>
          </template>
          </el-table-column>
        </el-table>
      </div>

      <Vbotton class = "uploadButton" slot="reference" :isWorking = "isUploadEnd" :clickMethod = "handleClick" :nameForButton = "'上传'" :isIcon ="true" :iconClass = "'bi-cloud-upload'"></Vbotton>
    </el-popover>
    <UploadDialog></UploadDialog>
  </div>

</template>

<script>
import Vbotton from "@/components/V-botton.vue"
import PathFinder from "@/components/PathFinder.vue"
import UploadDialog from "@/components/UploadDialog";
import mixinMethod from "@/assets/methods/UpAndDownload.vue";
export default {
  name: "Upload",
  mixins:[mixinMethod],
  data(){
    return{
      uploadVisible : false,
      uploadingData : [],
      uploadPath:"",
      fileType: "folder-fill",
      totalProgress:100,
      isUploadEnd:true,
    }
  },
  components:{
    Vbotton,
    PathFinder,
    UploadDialog,
  },
  methods:{
    center({row, column, rowIndex, columnIndex}){
      if(columnIndex !== 4){
        return  {background:'#1e1f26',color:'white'};
      }else if (columnIndex === 4){
        return  {background:'#1e1f26',color:'white','text-align' : "center"};
      }
    },

    handleClick(){
      this.$store.commit("updateState/updateVisable", true);
    },

    routeToUpload(){
      if("/mainpage/upload" !== this.$route.fullPath){
        this.$store.commit("updatePath","/mainpage/upload");
        //this.$store.commit("updateLastPath",this.$store.state.currentPath);
        this.$router.push("/mainpage/upload");
      }
    },

  },
  computed: {
    monitor () {

      return this.$store.state.updateState.preparedFiles.concat(this.$store.state.updateState.uploadingFiles).concat(this.$store.state.updateState.pausedFiles).concat(this.$store.state.updateState.waitingFiles);
    }
  },
  watch: {
    monitor:{
      handler: function (){
        this.uploadingData =  this.$store.state.updateState.preparedFiles.concat(this.$store.state.updateState.uploadingFiles).concat(this.$store.state.updateState.pausedFiles).concat(this.$store.state.updateState.waitingFiles);
      },
      //deep:true,
      immediate:true,
    },

    //监听数组complete 用于总进度条
    'uploadingData':{
      handler (val) {
        if(val.length !== 0){
          this.isUploadEnd = false;
          this.totalProgress = 0;
          for(let element of val){
            this.totalProgress = this.totalProgress + element.complete;
          }
          this.totalProgress = parseFloat((this.totalProgress / val.length).toFixed(1));

          if(this.totalProgress == 100){
            this.isUploadEnd=true;
          }
        }else{
          this.totalProgress = 100;
          this.isUploadEnd=true;
        }

      },
      // 这里是关键，代表递归监听 demo 的变化
      deep: true

    }


  },

  mounted() {

  }
}
</script>

<style scoped>

.popButton{
  position: absolute;
  margin: 4px;
  right: 0;
  top: 0;
  font-family: "AaGothic (Non-Commercial Use)";
}
.container{
  position: relative;
  width: 100%;
  height: 400px;

}
/deep/ .uploadButton.button-container{
  width: 130px;
}

/deep/.uploadButton.button-container::before {
  width: 90px;
}

.tmpTable{
  position: absolute;
  width: 100%;
  background: #1e1f26;
}

.container >>> .el-table--enable-row-hover .el-table__body tr:hover>td {
  background-color: #A85DC3;

}

.container >>> .el-table th.gutter{   /*解决el-table加了gutter后 边框出现白边*/
  background:#1e1f26;
}

.container >>> .el-table__row>td,.container >>>  .el-table th.is-leaf{
  border: none;
}
.container .el-table::before{
  height: 0;
}


/deep/ .el-table__body-wrapper::-webkit-scrollbar {
    width: 6px;
    height: 6px;

}

/deep/ .el-table__body-wrapper::-webkit-scrollbar-thumb {
   background-color: #a49f9f;
   border-radius: 3px;
}

</style>

