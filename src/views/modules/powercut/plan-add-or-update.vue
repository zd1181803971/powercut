<template>
  <el-dialog
    :title="'详细页面'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" ref="dataForm" label-width="130px">
    <el-form-item label="单位名称:" prop="company">
      {{dataForm.company}}
    </el-form-item>
    <el-form-item label="计划停电时间:">
      {{dataForm.blackoutTime}}
    </el-form-item>
    <el-form-item label="计划恢复时间:">
      {{dataForm.recoveryTime}}
    </el-form-item>
    <el-form-item label="影响台区数量:">
      {{dataForm.districtCount}}
    </el-form-item>
    <el-form-item label="影响用户数:">
      {{dataForm.userCount}}
    </el-form-item>
    <el-form-item label="停电原因:">
      {{dataForm.reason}}
    </el-form-item>
    <el-form-item label="近两个月停电次数:">
      {{dataForm.blackoutCount}}
    </el-form-item>
    <el-form-item label="工作内容:">
      {{dataForm.jobContent}}
    </el-form-item>
      <h4>计划停电台区信息</h4>
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
        dataList: [],
        dataListLoading: false,
        dataForm: {
          id: 0,
          company: '',
          blackoutTime: '',
          recoveryTime: '',
          districtCount: '',
          userCount: '',
          reason: '',
          blackoutCount: '',
          jobContent: ''
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
              url: this.$http.adornUrl(`/powercut/plan/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.company = data.plan.company
                this.dataForm.blackoutTime = data.plan.blackoutTime
                this.dataForm.recoveryTime = data.plan.recoveryTime
                this.dataForm.districtCount = data.plan.districtCount
                this.dataForm.userCount = data.plan.userCount
                this.dataForm.reason = data.plan.reason
                this.dataForm.blackoutCount = data.plan.blackoutCount
                this.dataForm.jobContent = data.plan.jobContent
                this.dataList = data.plan.districtDtoList
                console.log(data)
              }
            })
          }
        })
      }
    }
  }
</script>
