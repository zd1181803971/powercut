<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="单位名称" prop="company">
      <el-input v-model="dataForm.company" placeholder="单位名称"></el-input>
    </el-form-item>
    <el-form-item label="登陆账号" prop="userAccount">
      <el-input v-model="dataForm.userAccount" placeholder="登陆账号"></el-input>
    </el-form-item>
    <el-form-item label="登陆密码" prop="userPassword">
      <el-input v-model="dataForm.userPassword" placeholder="登陆密码"></el-input>
    </el-form-item>
    <el-form-item label="联系人" prop="userName">
      <el-input v-model="dataForm.userName" placeholder="联系人"></el-input>
    </el-form-item>
    <el-form-item label="联系方式" prop="phone">
      <el-input v-model="dataForm.phone" placeholder="联系方式"></el-input>
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
          userAccount: '',
          userPassword: '',
          userName: '',
          phone: '',
          gmtCreate: '',
          gmtModified: ''
        },
        dataRule: {
          company: [
            { required: true, message: '单位名称不能为空', trigger: 'blur' }
          ],
          userAccount: [
            { required: true, message: '登陆账号不能为空', trigger: 'blur' }
          ],
          userPassword: [
            { required: true, message: '登陆密码不能为空', trigger: 'blur' }
          ],
          userName: [
            { required: true, message: '联系人不能为空', trigger: 'blur' }
          ],
          phone: [
            { required: true, message: '联系方式不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/powercut/company/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.company = data.company.company
                this.dataForm.userAccount = data.company.userAccount
                this.dataForm.userPassword = data.company.userPassword
                this.dataForm.userName = data.company.userName
                this.dataForm.phone = data.company.phone
                this.dataForm.gmtCreate = data.company.gmtCreate
                this.dataForm.gmtModified = data.company.gmtModified
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
              url: this.$http.adornUrl(`/powercut/company/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'company': this.dataForm.company,
                'userAccount': this.dataForm.userAccount,
                'userPassword': this.dataForm.userPassword,
                'userName': this.dataForm.userName,
                'phone': this.dataForm.phone,
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
