<template>
  <el-dialog
    :title="'详细页面'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" ref="dataForm" label-width="140px">
      <el-form-item label="分类：">
          <span v-if="dataForm.category ===0">计划停电</span>
          <span v-if="dataForm.category ===1">临时停电</span>
      </el-form-item>
      <el-form-item label="单位名称：">
        {{dataForm.company}}
      </el-form-item>
      <el-form-item label="计划停电时间：">
        {{dataForm.blackoutTime}}
      </el-form-item>
      <el-form-item label="计划恢复时间：">
        {{dataForm.recoveryTime}}
      </el-form-item>
      <el-form-item label="影响台区数量：">
        {{dataForm.districtCount}}
      </el-form-item>
      <el-form-item label="影响用户数：">
        {{dataForm.userCount}}
      </el-form-item>
      <el-form-item label="停电原因：">
        {{dataForm.reason}}
      </el-form-item>
      <el-form-item label="近两个月停电次数：">
        {{dataForm.blackoutCount}}
      </el-form-item>
      <el-form-item label="工作内容：">
        {{dataForm.jobContent}}
      </el-form-item>
      <h4>计划停电台区信息：</h4>
      <el-table
        :data="dataList"
        border
        v-loading="dataListLoading"
        style="width: 100%;">
        <el-table-column
          prop="id"
          header-align="center"
          align="center"
          label="id">
        </el-table-column>
        <el-table-column
          prop="districtName"
          header-align="center"
          align="center"
          label="台区名称">
        </el-table-column>
        <el-table-column
          prop="manager"
          header-align="center"
          align="center"
          label="台区经理">
        </el-table-column>
        <el-table-column
          prop="userCount"
          header-align="center"
          align="center"
          label="用户数量">
        </el-table-column>
      </el-table>
      <el-form-item label="审批意见：" prop="部门审批意见">
        <el-input type="textarea"   :rows="2" v-model="dataForm.departmentOpinion" placeholder="手输，多行文本，200字以内。"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">返回</el-button>
      <el-button type="primary" @click="dataFormSubmit()">审批</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data () {
    return {
      visible: false,
      dataList: [],
      dataListLoading: false,
      dataForm: {
        id: 0,
        category: '',
        company: '',
        blackoutTime: '',
        recoveryTime: '',
        districtCount: '',
        userCount: '',
        reason: '',
        blackoutCount: '',
        jobContent: '',
        departmentOpinion: ''
      }
    }
  },
  computed: {
    userName: {
      get () { return this.$store.state }
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
            url: this.$http.adornUrl(`/powercut/plan/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.company = data.plan.company
              this.dataForm.category = data.plan.category
              this.dataForm.blackoutTime = data.plan.blackoutTime
              this.dataForm.recoveryTime = data.plan.recoveryTime
              this.dataForm.districtCount = data.plan.districtCount
              this.dataForm.userCount = data.plan.userCount
              this.dataForm.reason = data.plan.reason
              this.dataForm.blackoutCount = data.plan.blackoutCount
              this.dataForm.jobContent = data.plan.jobContent
              this.dataForm.departmentOpinion = data.plan.opinion
              this.dataList = data.plan.districtDtoList
            }
          })
        }
      })
    },
// 表单提交
    dataFormSubmit () {
      console.log(this.userName)
      // eslint-disable-next-line no-unused-vars
      var dawf
      if (this.userName === 'admin') {
        dawf = 2
      }
      this.$http({
        url: this.$http.adornUrl(`/powercut/plan/update`),
        method: 'post',
        data: this.$http.adornData({
          'id': this.dataForm.id,
          'opinion': this.dataForm.departmentOpinion,
          'planState': dawf
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.visible = false
              this.$emit('refreshDataList')
            }
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    }
  }
}
</script>
