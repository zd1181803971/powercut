<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="标题名" prop="title">
      <el-input v-model="dataForm.title" placeholder="标题名"></el-input>
    </el-form-item>
    <el-form-item label="时间" prop="newsTime">
      <el-input v-model="dataForm.newsTime" placeholder="时间"></el-input>
    </el-form-item>
    <el-form-item label="内容" prop="newsContent">
      <el-input v-model="dataForm.newsContent" placeholder="内容"></el-input>
    </el-form-item>
    <el-form-item label="状态" prop="state">
      <el-input v-model="dataForm.state" placeholder="状态"></el-input>
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
          title: '',
          newsTime: '',
          newsContent: '',
          state: '',
          gmtCreate: '',
          gmtModified: ''
        },
        dataRule: {
          title: [
            { required: true, message: '标题名不能为空', trigger: 'blur' }
          ],
          newsTime: [
            { required: true, message: '时间不能为空', trigger: 'blur' }
          ],
          newsContent: [
            { required: true, message: '内容不能为空', trigger: 'blur' }
          ],
          state: [
            { required: true, message: '状态不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/powercut/news/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.title = data.news.title
                this.dataForm.newsTime = data.news.newsTime
                this.dataForm.newsContent = data.news.newsContent
                this.dataForm.state = data.news.state
                this.dataForm.gmtCreate = data.news.gmtCreate
                this.dataForm.gmtModified = data.news.gmtModified
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
              url: this.$http.adornUrl(`/powercut/news/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'title': this.dataForm.title,
                'newsTime': this.dataForm.newsTime,
                'newsContent': this.dataForm.newsContent,
                'state': this.dataForm.state,
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
