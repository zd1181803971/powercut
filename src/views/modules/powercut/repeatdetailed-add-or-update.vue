<template>
  <el-dialog
    :title="'详细页面'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm"  ref="dataForm"  label-width="120px">
    <el-form-item label="单位名称:">
      {{dataForm.company}}
    </el-form-item>
    <el-form-item label="线路名称:">
      {{dataForm.lineRoadName}}
    </el-form-item>
    <el-form-item label="用户名称:">
      {{dataForm.userName}}
    </el-form-item>
    <el-form-item label="用户性质:">
      {{dataForm.userNatrue}}
    </el-form-item>
    <el-form-item label="起始时间:">
      {{dataForm.startTime}}
    </el-form-item>
    <el-form-item label="终止时间:">
      {{dataForm.stopTime}}
    </el-form-item>
    <el-form-item label="停电时户数:">
      {{dataForm.hourCount}}
    </el-form-item>
    <el-form-item label="重复停电次数：">
      {{dataForm.repeatCount}}
    </el-form-item>
    <el-form-item label="是否整改完成：">
      <div v-if="dataForm.isCorrectiveAction === null">
      </div>
      <div v-else-if="dataForm.isCorrectiveAction === 0">
        否
      </div>
      <div v-else-if="dataForm.isCorrectiveAction === 1">
        是
      </div>
    </el-form-item>
    <el-form-item label="沟通回访：">
      {{dataForm.communicate}}
    </el-form-item>
    <el-form-item label="停电原因：">
      {{dataForm.reason}}
    </el-form-item>
    <el-form-item label="停电分类：">
      {{dataForm.category}}
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          company: '',
          lineRoadName: '',
          userName: '',
          userNatrue: '',
          startTime: '',
          stopTime: '',
          hourCount: '',
          repeatCount: '',
          isCorrectiveAction: '',
          communicate: '',
          reason: '',
          category: ''
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/powercut/repeatdetailed/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.company = data.repeatDetailed.company
                this.dataForm.lineRoadName = data.repeatDetailed.lineRoadName
                this.dataForm.userName = data.repeatDetailed.userName
                this.dataForm.userNatrue = data.repeatDetailed.userNatrue
                this.dataForm.startTime = data.repeatDetailed.startTime
                this.dataForm.stopTime = data.repeatDetailed.stopTime
                this.dataForm.hourCount = data.repeatDetailed.hourCount
                this.dataForm.repeatCount = data.repeatDetailed.repeatCount
                this.dataForm.isCorrectiveAction = data.repeatDetailed.isCorrectiveAction
                this.dataForm.communicate = data.repeatDetailed.communicate
                this.dataForm.reason = data.repeatDetailed.reason
                this.dataForm.category = data.repeatDetailed.category
              }
            })
          }
        })
      }
    }
  }
</script>
