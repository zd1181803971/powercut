<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="天数" prop="days">
      <el-input v-model="dataForm.days" placeholder="天数"></el-input>
    </el-form-item>
    <el-form-item label="停电次数" prop="number">
      <el-input v-model="dataForm.number" placeholder="停电次数"></el-input>
    </el-form-item>
    <el-form-item label="数据创建时间" prop="gmtCreate">
      <el-input v-model="dataForm.gmtCreate" placeholder="数据创建时间"></el-input>
    </el-form-item>
    <el-form-item label="数据修改时间" prop="gmtModified">
      <el-input v-model="dataForm.gmtModified" placeholder="数据修改时间"></el-input>
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
          days: '',
          number: '',
          gmtCreate: '',
          gmtModified: ''
        },
        dataRule: {
          days: [
            { required: true, message: '天数不能为空', trigger: 'blur' }
          ],
          number: [
            { required: true, message: '停电次数不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/powercut/repeatrule/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.days = data.repeatrule.days
                this.dataForm.number = data.repeatrule.number
                this.dataForm.gmtCreate = data.repeatrule.gmtCreate
                this.dataForm.gmtModified = data.repeatrule.gmtModified
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
              url: this.$http.adornUrl(`/powercut/repeatrule/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'days': this.dataForm.days,
                'number': this.dataForm.number,
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
