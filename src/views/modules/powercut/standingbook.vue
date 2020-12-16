<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm"  @keyup.enter.native="getDataList()">
      <el-form-item>
        <span>
          单位名称：
        </span>
        <el-input
          placeholder="请输入单位名称"
          v-model="dataForm.station"
          clearable>
        </el-input>
      </el-form-item>
      <el-form-item>
        <span>
        停电时间:
      </span>
     <div class="block">
          <el-date-picker
            v-model="dataForm.timeList"
            type="datetimerange"
            :picker-options="dataForm.pickerOptions"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            align="right">
          </el-date-picker>
        </div>
      </el-form-item>

      <el-form-item>
        <span>
          停电原因:
        </span>
        <div>
          <el-select v-model="dataForm.reason" placeholder="请选择">
            <el-option
              v-for="item in dataList"
              :key="item.reason"
              :label="item.reason"
              :value="item.reason">
            </el-option>
          </el-select>
        </div>
      </el-form-item>

      <el-form-item>
        <span>
          台区经理:
        </span>
        <el-input v-model="dataForm.manager" placeholder="请输入台区经理"></el-input>
      </el-form-item>
    </el-form>

    <el-form>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button @click="clear()">清空</el-button>
        <el-button @click="exprtStandingBook()">导出</el-button>
        <el-button v-if="isAuth('powercut:standingbook:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>

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
        label="id">
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="单位名称">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">
           {{scope.row.company}}
          </el-button>
        </template>
      </el-table-column>
      <el-table-column
        prop="blackoutTime"
        header-align="center"
        align="center"
        label="停电时间">
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="停电时长">
        <template slot-scope="scope">
          <i class="el-icon-time"></i>
          <span style="margin-left: 10px">{{ scope.row.blackoutDuration }}小时</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="districtCount"
        header-align="center"
        align="center"
        label="影响台区数量">
      </el-table-column>
      <el-table-column
        prop="userCount"
        header-align="center"
        align="center"
        label="影响用户数">
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="是否计划内">
        <template slot-scope="scope">
          <span v-if="scope.row.isPlan == 0">否</span>
          <span v-if="scope.row.isPlan == 1">是</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="reason"
        header-align="center"
        align="center"
        label="停电原因">
      </el-table-column>
      <el-table-column
        prop="blackoutCount"
        header-align="center"
        align="center"
        label="近两个月停电次数">
      </el-table-column>

    </el-table>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>

<!--    分页组件-->
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>

  </div>
</template>

<script>
  import AddOrUpdate from './standingbook-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          // 单位名称
          station: '',
          // 台区经理
          manager: '',
          // pickerOptions日期时间
          pickerOptions: {
            shortcuts: [{
              text: '最近一周',
              onClick (picker) {
                const end = new Date()
                const start = new Date()
                start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
                picker.$emit('pick', [start, end])
              }
            }, {
              text: '最近一个月',
              onClick (picker) {
                const end = new Date()
                const start = new Date()
                start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
                picker.$emit('pick', [start, end])
              }
            }, {
              text: '最近三个月',
              onClick (picker) {
                const end = new Date()
                const start = new Date()
                start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
                picker.$emit('pick', [start, end])
              }
            }]
          },
          timeList: '',
          reason: ''
        },
        ids: 1,
        addOrUpdateVisible: false,
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        test: []
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // tobecontinue
      // exprtStandingBook () {
      //   this.dataListLoading = true
      //   this.$http({
      //     url: this.$http.adornUrl('/powercut/standingbook/exprtStandingBook'),
      //     method: 'get',
      //     params: this.$http.adornParams({
      //     })
      //   }).then(({data}) => {
      //     if (data && data.code === 0) {
      //     } else {
      //     }
      //   })
      // },
      clear () {
        this.dataForm.station = ''
        this.dataForm.timeList = ''
        this.dataForm.reason = ''
        this.dataForm.manager = ''
        this.getDataList()
      },
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/powercut/standingbook/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'company': this.dataForm.station || null,
            'startTime': this.dataForm.timeList[0] || null,
            'stopTime': this.dataForm.timeList[1] || null,
            'reason': this.dataForm.reason || null,
            'manager': this.dataForm.manager || null
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
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },

      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/powercut/standingbook/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }

  }
</script>
