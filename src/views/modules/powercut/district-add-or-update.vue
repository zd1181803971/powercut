<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="130px">
    <el-form-item label="单位名称:" prop="company">
      <el-input v-model="dataForm.company" placeholder="单位名称"></el-input>
    </el-form-item>
    <el-form-item label="变电站名称:" prop="stationName">
      <el-input v-model="dataForm.stationName" placeholder="变电站名称"></el-input>
    </el-form-item>
    <el-form-item label="线路名称:" prop="lineRoadName">
      <el-input v-model="dataForm.lineRoadName" placeholder="线路名称"></el-input>
    </el-form-item>
    <el-form-item label="线路名称:" prop="lineSegmentName">
      <el-input v-model="dataForm.lineSegmentName" placeholder="线路名称"></el-input>
    </el-form-item>
    <el-form-item label="台区用户名称:" prop="userName">
      <el-input v-model="dataForm.userName" placeholder="台区用户名称"></el-input>
    </el-form-item>
    <el-form-item label="用户性质:" prop="userNatrue">
      <template>
        <el-radio v-model="dataForm.userNatrue" label="0">公用</el-radio>
        <el-radio v-model="dataForm.userNatrue" label="1">专用</el-radio>
      </template>
    </el-form-item>
    <el-form-item label="台区经理:" prop="manager">
      <el-input v-model="dataForm.manager" placeholder="台区经理"></el-input>
    </el-form-item>
    <el-form-item label="用户数量:" prop="userCount">
      <el-input v-model="dataForm.userCount" placeholder="用户数量"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
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
        },
        dataRule: {
          company: [
            { required: true, message: '单位名称不能为空', trigger: 'blur' }
          ],
          stationName: [
            { required: true, message: '变电站名称不能为空', trigger: 'blur' }
          ],
          lineRoadName: [
            { required: true, message: '线路名称不能为空', trigger: 'blur' }
          ],
          lineSegmentName: [
            { required: true, message: '线路名称不能为空', trigger: 'blur' }
          ],
          userName: [
            { required: true, message: '台区用户名称不能为空', trigger: 'blur' }
          ],
          userNatrue: [
            { required: true, message: '用户性质不能为空', trigger: 'blur' }
          ],
          manager: [
            { required: true, message: '台区经理不能为空', trigger: 'blur' }
          ],
          userCount: [
            { required: true, message: '用户数量不能为空', trigger: 'blur' }
          ]
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
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/powercut/district/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'company': this.dataForm.company,
                'stationName': this.dataForm.stationName,
                'lineRoadName': this.dataForm.lineRoadName,
                'lineSegmentName': this.dataForm.lineSegmentName,
                'userName': this.dataForm.userName,
                'userNatrue': this.dataForm.userNatrue,
                'manager': this.dataForm.manager,
                'userCount': this.dataForm.userCount,
                'gmtCreate': this.dataForm.gmtCreate,
                'gmtModified': this.dataForm.gmtModified
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
        })
      }
    }
  }
</script>
