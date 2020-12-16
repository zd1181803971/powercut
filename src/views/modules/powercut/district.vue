<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
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
          变电站名称：
        </span>
        <el-input
          placeholder="请输入变电站名称"
          v-model="dataForm.stationName"
          clearable>
        </el-input>
      </el-form-item>
      <el-form-item>
        <span>
          线路名称：
        </span>
        <el-input
          placeholder="请输入线路名称"
          v-model="dataForm.lineRoadName"
          clearable>
        </el-input>
      </el-form-item>
      <el-form-item>
        <span>
          线段名称：
        </span>
        <el-input
          placeholder="请输入线段名称"
          v-model="dataForm.lineSegmentName"
          clearable>
        </el-input>
      </el-form-item>
      <el-form-item>
        <span>
          台区用户名称：
        </span>
        <el-input
          placeholder="请输入台区用户名称"
          v-model="dataForm.userName"
          clearable>
        </el-input>
      </el-form-item>
      <el-form-item>
        <span>
          台区经理：
        </span>
        <el-input
          placeholder="请输入台区经理"
          v-model="dataForm.manager "
          clearable>
        </el-input>
      </el-form-item>
      <br>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button @click="clear()">清空</el-button>
        <el-button @click="importDistrict()">导入</el-button>
        <el-button @click="">模板下载</el-button>
        <el-button v-if="isAuth('powercut:district:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
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
        prop="company"
        header-align="center"
        align="center"
        label="单位名称">
      </el-table-column>
      <el-table-column
        prop="stationName"
        header-align="center"
        align="center"
        label="变电站名称">
      </el-table-column>
      <el-table-column
        prop="lineRoadName"
        header-align="center"
        align="center"
        label="线路名称">
      </el-table-column>
      <el-table-column
        prop="lineSegmentName"
        header-align="center"
        align="center"
        label="线段名称">
      </el-table-column>
      <el-table-column
        prop="userName"
        header-align="center"
        align="center"
        label="台区用户名称">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="showHandle(scope.row.id)">
            {{scope.row.userName}}
          </el-button>
        </template>
      </el-table-column>
      <el-table-column
        prop="userNatrue"
        header-align="center"
        align="center"
        label="用户性质">
        <template slot-scope="scope">
          <span v-if="scope.row.userNatrue">公用</span>
          <span v-if="!scope.row.userNatrue">专用</span>
        </template>
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
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
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
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <show-page v-if="showVisible" ref="showPage" @refreshDataList="getDataList"></show-page>
  </div>
</template>

<script>
  import AddOrUpdate from './district-add-or-update'
  import ShowPage from './district-show'
  export default {
    data () {
      return {
        dataForm: {
          station: '',
          stationName: '',
          lineRoadName: '',
          lineSegmentName: '',
          userName: '',
          manager: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        showVisible: false
      }
    },
    components: {
      AddOrUpdate, ShowPage

    },
    activated () {
      this.getDataList()
    },
    methods: {
      // // 模板导入
      // importDistrict(){
      //   this.dataListLoading = true
      //   this.$http({
      //     url: this.$http.adornUrl('/powercut/district/importDistrict'),
      //     method: 'post',
      //     params: this.$http.adornParams({
      //
      //     })
      //   }).then(({data}) => {
      //     if (data && data.code === 0) {
      //
      //     } else {
      //
      //     }
      //     this.dataListLoading = false
      //   })
      // },
      clear () {
        this.dataForm.station = ''
        this.dataForm.stationName = ''
        this.dataForm.lineRoadName = ''
        this.dataForm.lineSegmentName = ''
        this.dataForm.userName = ''
        this.dataForm.manager = ''
        this.getDataList()
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/powercut/district/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'company': this.dataForm.station || null,
            'station_name': this.dataForm.stationName || null,
            'line_road_name': this.dataForm.lineRoadName || null,
            'line_segment_name': this.dataForm.lineSegmentName || null,
            'user_name': this.dataForm.userName || null,
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
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 展示详细信息
      showHandle (id) {
        this.showVisible = true
        this.$nextTick(() => {
          this.$refs.showPage.init(id)
        })
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
            url: this.$http.adornUrl('/powercut/district/delete'),
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
