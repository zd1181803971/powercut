<template>
  <el-dialog
    :title="'详情页面'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm"  ref="dataForm"  label-width="120px">
      <el-form-item label="单位名称:">
        {{dataForm.company}}
      </el-form-item>
      <el-form-item label="变电站名称:">
        {{dataForm.stationName}}
      </el-form-item>
      <el-form-item label="线路名称:">
        {{dataForm.lineRoadName}}
      </el-form-item>
      <el-form-item label="线段名称:">
        {{dataForm.lineSegmentName}}
      </el-form-item>
      <el-form-item label="台区用户名称:">
        {{dataForm.userName}}
      </el-form-item>
      <el-form-item label="用户性质:">
          <span v-if="dataForm.userNatrue">公用</span>
          <span v-if="!dataForm.userNatrue">专用</span>
      </el-form-item>
      <el-form-item label="台区经理:">
        {{dataForm.manager}}
      </el-form-item>
      <el-form-item label="用户数量:">
        {{dataForm.userCount}}
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
        stationName: '',
        lineRoadName: '',
        lineSegmentName: '',
        userName: '',
        userNatrue: '',
        manager: '',
        userCount: '',
        gmtCreate: '',
        gmtModified: ''
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
            url: this.$http.adornUrl(`/powercut/district/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.company = data.district.company
              this.dataForm.stationName = data.district.stationName
              this.dataForm.lineRoadName = data.district.lineRoadName
              this.dataForm.lineSegmentName = data.district.lineSegmentName
              this.dataForm.userName = data.district.userName
              this.dataForm.userNatrue = data.district.userNatrue
              this.dataForm.manager = data.district.manager
              this.dataForm.userCount = data.district.userCount
              this.dataForm.gmtCreate = data.district.gmtCreate
              this.dataForm.gmtModified = data.district.gmtModified
            }
          })
        }
      })
    }
  }
}
</script>
