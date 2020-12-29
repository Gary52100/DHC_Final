<template>
  <div class="mod-user">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.pageName" placeholder="页面名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-select v-model="dataForm.pageStatus" placeholder="请选择状态">
          <el-option label="全部" value="-1"></el-option>
          <el-option label="待发布" value="0"></el-option>
          <el-option label="已发布" value="1"></el-option>
          <el-option label="已下线" value="2"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-date-picker v-model="dataForm.createDate" align="right" type="date" placeholder="创建时间" :picker-options="pickerOptions"></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button type="primary" @click="H5Upload()">上传</el-button>
        <el-button type="danger" @click="H5Delete()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeUser" style="width: 100%;">
      <el-table-column type="selection" :selectable="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="id" header-align="center" align="center" width="80" label="页面ID"></el-table-column>
      <el-table-column prop="webpageName" header-align="center" align="center" label="页面名称"></el-table-column>
      <el-table-column prop="status" :formatter="formatterColumn" header-align="center" align="center" label="状态"></el-table-column>
      <el-table-column prop="createUser" header-align="center" align="center" label="创建者"></el-table-column>
      <el-table-column prop="createTime" :formatter="formatterColumnDate" header-align="center" align="center" label="创建时间"></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="200" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="H5Show(scope.row.id)" v-if="scope.row.status=='1'">浏览</el-button>
          <el-button type="text" size="small" @click="H5Edit(scope.row.id)" v-if="scope.row.status!='1'">编辑</el-button>
          <el-button type="text" size="small" @click="H5UpdateStatus(scope.row.id, scope.row.status)" v-if="scope.row.status=='0'">发布</el-button>
          <el-button type="text" size="small" @click="H5UpdateStatus(scope.row.id, scope.row.status)" v-if="scope.row.status=='1'">下线</el-button>
          <el-button type="text" size="small" @click="H5UpdateStatus(scope.row.id, scope.row.status)" v-if="scope.row.status=='2'">发布</el-button>
          <el-button type="text" size="small" @click="H5Delete(scope.row.id)" v-if="scope.row.status!='1'">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeUser"
      @current-change="currentChangeUser"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 授予权限 / 查看权限 -->
    <H5Show v-if="H5ShowVisible" ref="H5Show" @refreshDataList="getDataList"></H5Show>
    <H5Edit v-if="H5EditVisible" ref="H5Edit" @refreshDataList="getDataList"></H5Edit>
    <H5Upload v-if="H5UploadVisible" ref="H5Upload" @refreshDataList="getDataList"></H5Upload>
  </div>
</template>

<script>
  import H5Edit from './H5-edit'
  import H5Show from './H5-show'
  import H5Upload from './H5-upload'
  export default {
    data () {
      return {
        dataForm: {
          pageName: '',
          pageStatus: '-1',
          createDate: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        H5ShowVisible: false,
        H5EditVisible: false,
        H5UploadVisible: false,
        pickerOptions: {
          disabledDate (time) {
            return time.getTime() > Date.now()
          },
          shortcuts: [{
            text: '今天',
            onClick (picker) {
              picker.$emit('pick', new Date())
            }
          }, {
            text: '昨天',
            onClick (picker) {
              const date = new Date()
              date.setTime(date.getTime() - 3600 * 1000 * 24)
              picker.$emit('pick', date)
            }
          }, {
            text: '一周前',
            onClick (picker) {
              const date = new Date()
              date.setTime(date.getTime() - 3600 * 1000 * 24 * 7)
              picker.$emit('pick', date)
            }
          }]
        }
      }
    },
    components: {
      H5Show,
      H5Edit,
      H5Upload
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        let paramsDate = ''
        if (this.dataForm.createDate !== '' && this.dataForm.createDate !== null) {
          paramsDate = this.$moment(this.dataForm.createDate).format('YYYY-MM-DD')
        }
        this.$http({
          url: this.$http.adornUrl('/webpage/findListPage'),
          method: 'post',
          data: this.$http.adornData({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'webpage_name': this.dataForm.pageName,
            'create_time': paramsDate,
            'status': this.dataForm.pageStatus
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            console.log(data.page.list)
            this.dataList = data.page.list
            this.totalPage = data.page.total
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      selection (row, index) {
        if (row.status !== 1) {
          return true
        } else {
          return false
        }
      },
      // 格式化状态
      formatterColumn (row, column) {
        switch (row.status) {
          case 0:
            return '待发布'
          case 1:
            return '已发布'
          case 2:
            return '已下线'
        }
      },
      // 格式化时间
      formatterColumnDate (row, column) {
        var dateee = new Date(row.createTime).toJSON()
        return new Date(+new Date(dateee) + 8 * 3600 * 1000).toISOString().replace(/T/g, ' ').replace(/\.[\d]{3}Z/, '')
      },
      // 每页数
      sizeChangeUser (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeUser (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeUser (val) {
        this.dataListSelections = val
      },
      // 批量删除 or 删除
      H5Delete (id) {
        var pageIds = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${pageIds.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/webpage/deleteBatch'),
            method: 'post',
            data: this.$http.adornData({
              'idList': pageIds
            })
          }).then(({data}) => {
            if (data && data.code === 200) {
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
        }).catch(() => {})
      },
      // 上传
      H5Upload (id) {
        this.H5UploadVisible = true
        this.$nextTick(() => {
          this.$refs.H5Upload.init()
        })
      },
      H5UpdateStatus (id, status) {
        let pageStatus = status === 1 ? '2' : '1'
        this.$http({
          url: this.$http.adornUrl('/webpage/updateStatus'),
          method: 'post',
          params: this.$http.adornParams({
            'id': id,
            'status': pageStatus
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
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
      },
      // 浏览
      H5Show (id) {
        this.H5ShowVisible = true
        this.$nextTick(() => {
          this.$refs.H5Show.init(id)
        })
      },
      // 编辑
      H5Edit (id) {
        this.H5EditVisible = true
        this.$nextTick(() => {
          this.$refs.H5Edit.init(id)
        })
      }
    }
  }
</script>
