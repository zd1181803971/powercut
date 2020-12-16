<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="130px">
      <el-form-item label="报告名称:" prop="reportName">
        <el-input v-model="dataForm.reportName" placeholder="报告名称"></el-input>
      </el-form-item>
      <el-form-item label="备注:" prop="remarks">
        <el-input v-model="dataForm.remarks" placeholder="备注"></el-input>
      </el-form-item>
      <el-form-item label="统计开始时间:" prop="startTime">
<!--        <el-input v-model="dataForm.startTime" placeholder="统计开始时间"></el-input>-->
        <div class="block">
          <el-date-picker
            v-model="dataForm.startTime"
            type="datetime"
            placeholder="统计开始时间">
          </el-date-picker>
        </div>
      </el-form-item>
      <el-form-item label="统计结束时间:" prop="stopTime">
<!--        <el-input v-model="dataForm.stopTime" placeholder="统计结束时间"></el-input>-->
        <div class="block">
          <el-date-picker
            v-model="dataForm.stopTime"
            type="datetime"
            placeholder="统计结束时间">
          </el-date-picker>
        </div>
      </el-form-item>
      <el-form-item label="单位名称:" prop="station">
        <el-input v-model="dataForm.station" placeholder="单位名称"></el-input>
      </el-form-item>
      <el-form-item label="台区经理:" prop="manager">
        <el-input v-model="dataForm.manager" placeholder="台区经理"></el-input>
      </el-form-item>

      <el-table
        :data="dataList"
        border
        v-loading="dataListLoading"
        @selection-change="selectionChangeHandle"
        style="width: 100%;">
        <el-table-column
          type="selection"
          header-align="center"
          align="center"
          width="50">
        </el-table-column>
        <el-table-column
          prop="id"
          header-align="center"
          align="center"
          label="序号">
        </el-table-column>
        <el-table-column
          prop="company"
          header-align="center"
          align="center"
          label="单位名称">
        </el-table-column>
        <el-table-column
          prop="lineRoadName"
          header-align="center"
          align="center"
          label="线路名称">
        </el-table-column>
        <el-table-column
          prop="userName"
          header-align="center"
          align="center"
          label="用户名称">
        </el-table-column>
        <el-table-column
          prop="userNatrue"
          header-align="center"
          align="center"
          label="用户性质">
        </el-table-column>
        <el-table-column
          prop="startTime"
          header-align="center"
          align="center"
          label="起始时间">
        </el-table-column>
        <el-table-column
          prop="stopTime"
          header-align="center"
          align="center"
          label="终止时间">
        </el-table-column>
        <el-table-column
          prop="hourCount"
          header-align="center"
          align="center"
          label="停电时户数">
        </el-table-column>
        <el-table-column
          prop="repeatCount"
          header-align="center"
          align="center"
          label="重复停电次数">
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="sizeChangeHandle"
        @current-change="currentChangeHandle"
        :current-page="pageIndex"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="pageSize"
        :total="totalPage"
        layout="total, sizes, prev, pager, next, jumper">
      </el-pagination>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">下一步</el-button>
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
        reportName: '',
        startTime: '',
        stopTime: '',
        createTime: '',
        reportHref: '',
        remarks: '',
        gmtCreate: '',
        gmtModified: ''
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      dataRule: {
        reportName: [
          {required: true, message: '报告名称不能为空', trigger: 'blur'}
        ],
        startTime: [
          {required: true, message: '统计开始时间不能为空', trigger: 'blur'}
        ],
        stopTime: [
          {required: true, message: '统计结束时间不能为空', trigger: 'blur'}
        ],
        remarks: [
          {required: true, message: '备注不能为空', trigger: 'blur'}
        ]
      }
    }
  },
  methods: {
    // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getDataList()
    },
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/powercut/repeatdetailed/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list
          this.totalPage = data.page.totalCount
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    init (id) {
      this.getDataList()
      this.dataForm.id = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/powercut/report/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.reportName = data.report.reportName
              this.dataForm.startTime = data.report.startTime
              this.dataForm.stopTime = data.report.stopTime
              this.dataForm.createTime = data.report.createTime
              this.dataForm.reportHref = data.report.reportHref
              this.dataForm.remarks = data.report.remarks
              this.dataForm.gmtCreate = data.report.gmtCreate
              this.dataForm.gmtModified = data.report.gmtModified
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
            url: this.$http.adornUrl(`/powercut/report/${!this.dataForm.id ? 'save' : 'update'}`),
            method: 'post',
            data: this.$http.adornData({
              'id': this.dataForm.id || undefined,
              'reportName': this.dataForm.reportName,
              'startTime': this.dataForm.startTime,
              'stopTime': this.dataForm.stopTime,
              'createTime': this.dataForm.createTime,
              'reportHref': this.dataForm.reportHref,
              'remarks': this.dataForm.remarks,
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
    },
    // 多选
    selectionChangeHandle (val) {
      this.dataListSelections = val
    }
  }
}
</script>
