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
      <el-table-column label="评论id" width="100" align="center">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="用户id" width="100" align="center">
        <template slot-scope="{row}">
          <span>{{ row.userID }}</span>
        </template>
      </el-table-column>
      <el-table-column label="评论内容" width="800" align="center">
        <template slot-scope="{row}">
          <span>{{ row.content }}</span>
        </template>
      </el-table-column>
      <el-table-column label="评论时间" width="300" align="center">
        <template slot-scope="{row}">
          <span>{{ row.time }}</span>
        </template>
      </el-table-column>
      <el-table-column label="点赞数" width="100" align="center">
        <template slot-scope="{row}">
          <span>{{ row.good }}</span>
        </template>
      </el-table-column>
      <el-table-column label="评论类型" width="100" align="center">
        <template slot-scope="{row}">
          <span>{{ row.type }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-popconfirm
            confirm-button-text="确认"
            cancel-button-text="取消"
            icon="el-icon-info"
            icon-color="red"
            title="确认删除该评论么？"
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
      pageSize: 10 // 每一页的数量
    }
  },
  created() {
    this.getList() // 页面加载的时候获取用户信息
  },
  methods: {
    getList() { // 调用接口获取所有评论
      this.listLoading = true // 开启加载情况
      this.tools.requests(this.G.SERVER + '/api/admin/getAllComment', {}, 'get').then((response) => {
        if (response != null && response.code === 1) {
          this.list = response.data
          this.pageList = response.data.slice(0, 10)
          this.total = response.data.length // 数据总数
        } else {
          this.$message({
            message: '获取评论列表失败',
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
    handleDeleteUser(row) { // 删除评论
      console.log('删除打卡', row)
      this.tools.requests(this.G.SERVER + '/api/admin/comment/delete', {
        commentID: row.id
      }, 'post').then((response) => { // 调用删除评论
        if (response != null && response.code === 1) {
          const index = this.list.findIndex(v => v.id === row.id) // 查找下标
          this.list.splice(index, 1) // 模拟修改数据
          console.log(index, this.list, '删除评论')
          this.total = this.list.length // 删除评论后要修改这里
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
    }
  }
}
</script>
<style>
  .pageContain{
    margin-top: 2em;
  }
</style>
