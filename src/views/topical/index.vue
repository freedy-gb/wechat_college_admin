<template>
  <div class="app-container">
    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="pageList"
      border
      fit
      highlight-current-row
      style="width: 100%;"
    >
      <el-table-column label="话题id" width="80" align="center">
        <template slot-scope="{row}">
          <span>{{ row.topicalID }}</span>
        </template>
      </el-table-column>
      <el-table-column label="作者id" width="80" align="center">
        <template slot-scope="{row}">
          <span>{{ row.userID }}</span>
        </template>
      </el-table-column>
      <el-table-column label="作者头像" width="100" align="center">
        <template slot-scope="{row}">
          <el-avatar :size="50" :src="row.imgUrl" />
        </template>
      </el-table-column>
      <el-table-column label="话题标题" width="360" align="center">
        <template slot-scope="{row}">
          <span>{{ row.title }}</span>
        </template>
      </el-table-column>
      <el-table-column label="发布时间" width="360" align="center">
        <template slot-scope="{row}">
          <span>{{ row.time }}</span>
        </template>
      </el-table-column>
      <el-table-column label="浏览次数" width="100" align="center">
        <template slot-scope="{row}">
          <span>{{ row.view }}</span>
        </template>
      </el-table-column>
      <el-table-column label="点赞次数" width="100" align="center">
        <template slot-scope="{row}">
          <span>{{ row.good }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-popconfirm
            confirm-button-text="确认"
            cancel-button-text="取消"
            icon="el-icon-info"
            icon-color="red"
            title="确认删除该话题么？"
            @onConfirm="handleDeleteUser(row)"
          >
            <el-button slot="reference" size="mini" type="danger">删除</el-button>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      class="pageContain"
      :total="total"
      :page-size="pageSize"
      layout="total, prev, pager, next, jumper"
      :current-page="currentPage"
      @current-change="handleCurrentChange"
    />

    <!--    // 编辑用户信息-->
    <el-dialog title="修改话题内容" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :model="temp" label-position="left" label-width="70px" style="width: 70%; margin-left:50px;">
        <el-form-item label="标题" prop="type">
          <el-input v-model="temp.title" />
        </el-form-item>
        <el-form-item label="内容" prop="timestamp">
          <el-input v-model="temp.content" type="textarea" autosize />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          取消
        </el-button>
        <el-button type="primary" @click="updateData()">
          确认
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>

export default {
  name: 'ComplexTable',
  data() {
    return {
      tableKey: 0,
      list: null, // 存储用户信息
      pageList: null, // 存储当前页面的用户信息
      total: 0, // 用于分页的显示总数
      listLoading: true, // 是否显示加载状态
      temp: {// 编辑的时候的板块
      },
      currentPage: 1, // 当前页码
      pageSize: 10, // 每一页的数量
      dialogFormVisible: false, // 是否展示弹框
      downloadLoading: false
    }
  },
  created() {
    this.getList() // 页面加载的时候获取用户信息
  },
  methods: {
    getList() {
      this.listLoading = true // 开启加载情况
      this.tools.requests(this.G.SERVER + '/api/admin/topical/get/topicalList', {}, 'get').then((response) => { // 调用接口获取话题信息
        if (response != null && response.code === 1) {
          this.list = response.data
          this.pageList = response.data.slice(0, 10)
          this.total = response.data.length // 数据总数
        } else {
          this.$message({
            message: '获取话题列表失败',
            type: 'fail'
          })
        }
        this.listLoading = false // 关闭加载
      })
    },
    handleCurrentChange(val) { // 分页相关
      this.currentPage = val // 存储当前页面
      console.log('当前页:', val)
      this.pageList = this.list.slice((val - 1) * 10, val * 10)
    },
    handleDeleteUser(row) { // 删除话题
      this.tools.requests(this.G.SERVER + '/api/topical/delete', {
        userId: row.userID,
        topicalID: row.topicalID
      }, 'post').then((response) => { // 调用接口获取用户信息
        if (response != null && response.code === 1) {
          const index = this.list.findIndex(v => v.topicalID === row.topicalID) // 查找下标
          this.list.splice(index, 1) // 模拟修改数据
          this.total = this.list.length // 删除了话题后要修改这里
          this.pageList = this.list.slice((this.currentPage - 1) * 10, this.currentPage * 10)
          this.$message({
            message: '删除成功',
            type: 'success'
          })
        } else {
          this.$message({
            message: '删除失败',
            type: 'fail'
          })
        }
        this.dialogFormVisible = false // 将弹出框隐藏
      })
    },
    handleUpdate(row) { // 修改话题信息
      console.log('每一行数据', row)
      this.temp = row
      console.log('temp', this.temp)
      this.temp.timestamp = new Date(this.temp.timestamp)
      this.dialogFormVisible = true // 开启弹框
    },
    updateData() { // 更新话题信息
      const temp = this.temp
      this.tools.requests(this.G.SERVER + '/api/admin/topical/edit', {
        topicalID: temp.topicalID,
        title: temp.title,
        content: temp.content
      }, 'post').then((response) => { // 调用接口获取用户信息
        if (response != null && response.code === 1) {
          const index = this.list.findIndex(v => v.topicalID === this.temp.topicalID) // 查找下标
          this.list.splice(index, 1, this.temp) // 模拟修改数据
          this.$notify({
            title: '成功',
            message: '更新成功',
            type: 'success',
            duration: 1000
          })
        } else {
          this.$message({
            message: '修改话题信息失败',
            type: 'fail'
          })
        }
        this.dialogFormVisible = false // 将弹出框隐藏
      })
    }
  }
}
</script>
<style>
  .pageContain{
    margin-top: 2em;
  }
</style>
