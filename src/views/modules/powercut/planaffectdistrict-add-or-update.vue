<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="台区Id" prop="districtId">
      <el-input v-model="dataForm.districtId" placeholder="台区Id"></el-input>
    </el-form-item>
    <el-form-item label="台区名称" prop="districtName">
      <el-input v-model="dataForm.districtName" placeholder="台区名称"></el-input>
    </el-form-item>
    <el-form-item label="停电台账Id" prop="standingbookId">
      <el-input v-model="dataForm.standingbookId" placeholder="停电台账Id"></el-input>
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
          districtId: '',
          districtName: '',
          standingbookId: '',
          gmtCreate: '',
          gmtModified: ''
        },
        dataRule: {
          districtId: [
            { required: true, message: '台区Id不能为空', trigger: 'blur' }
          ],
          districtName: [
            { required: true, message: '台区名称不能为空', trigger: 'blur' }
          ],
          standingbookId: [
            { required: true, message: '停电台账Id不能为空', trigger: 'blur' }
          ],
          gmtCreate: [
            { required: true, message: '数据创建时间不能为空', trigger: 'blur' }
          ],
          gmtModified: [
            { required: true, message: '数据修改时间不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/powercut/planaffectdistrict/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.districtId = data.planAffectDistrict.districtId
                this.dataForm.districtName = data.planAffectDistrict.districtName
                this.dataForm.standingbookId = data.planAffectDistrict.standingbookId
                this.dataForm.gmtCreate = data.planAffectDistrict.gmtCreate
                this.dataForm.gmtModified = data.planAffectDistrict.gmtModified
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
              url: this.$http.adornUrl(`/powercut/planaffectdistrict/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'districtId': this.dataForm.districtId,
                'districtName': this.dataForm.districtName,
                'standingbookId': this.dataForm.standingbookId,
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
