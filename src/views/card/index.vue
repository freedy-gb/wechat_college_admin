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
      <el-table-column label="打卡id" width="100" align="center">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="发布人id" width="100" align="center">
        <template slot-scope="{row}">
          <span>{{ row.userID }}</span>
        </template>
      </el-table-column>
      <el-table-column label="作者头像" width="100" align="center">
        <template slot-scope="{row}">
          <el-avatar :size="50" :src="row.imgUrl" />
        </template>
      </el-table-column>
      <el-table-column label="发布人名字" width="300" align="center">
        <template slot-scope="{row}">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="打卡标题" width="300" align="center">
        <template slot-scope="{row}">
          <span>{{ row.title }}</span>
        </template>
      </el-table-column>
      <el-table-column label="发布时间" width="300" align="center">
        <template slot-scope="{row}">
          <span>{{ row.time }}</span>
        </template>
      </el-table-column>
      <el-table-column label="需坚持天数" width="100" align="center">
        <template slot-scope="{row}">
          <span>{{ row.totalDay }}</span>
        </template>
      </el-table-column>
      <el-table-column label="加入人数" width="100" align="center">
        <template slot-scope="{row}">
          <span>{{ row.peoples }}</span>
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
    <el-dialog title="修改打卡内容" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :model="temp" label-position="left" label-width="70px" style="width: 70%; margin-left:50px;">
        <el-form-item label="标题" prop="type">
          <el-input v-model="temp.title" />
        </el-form-item>
        <el-form-item label="信息" prop="timestamp">
          <el-input v-model="temp.description" type="textarea" autosize />
        </el-form-item>
        <el-form-item label="总的天数" prop="type">
          <el-input v-model="temp.totalDay" />
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
      this.tools.requests(this.G.SERVER + '/api/card/get/cardList', { userID: 0 }, 'get').then((response) => { // 调用接口获取打卡信息
        if (response != null && response.code === 1) {
          this.list = response.data
          this.pageList = response.data.slice(0, 10)
          this.total = response.data.length // 数据总数
        } else {
          this.$message({
            message: '获取打卡列表失败',
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
    handleDeleteUser(row) { // 删除打卡
      console.log('删除打卡', row)
      this.tools.requests(this.G.SERVER + '/api/card/delete', {
        cardID: row.id,
        userID: row.userID
      }, 'post').then((response) => { // 调用删除打卡
        if (response != null && response.code === 1) {
          const index = this.list.findIndex(v => v.id === row.id) // 查找下标
          this.list.splice(index, 1) // 模拟修改数据
          this.total = this.list.length // 删除打卡后要修改这里
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
    handleUpdate(row) { // 修改打卡信息
      console.log('每一行数据', row)
      this.temp = row
      console.log('temp', this.temp)
      this.temp.timestamp = new Date(this.temp.timestamp)
      this.dialogFormVisible = true // 开启弹框
    },
    updateData() { // 更新打卡信息
      const temp = this.temp
      console.log(this.temp, '更新打卡信息')
      this.tools.requests(this.G.SERVER + '/api/admin/card/change/cardInfo', {
        title: temp.title,
        cardID: temp.id,
        content: temp.description,
        totalDay: temp.totalDay
      }, 'post').then((response) => { // 更新打卡信息
        if (response != null && response.code === 1) {
          const index = this.list.findIndex(v => v.cardID === this.temp.cardID) // 查找下标
          this.list.splice(index, 1, this.temp) // 模拟修改数据
          this.$notify({
            title: '成功',
            message: '更新成功',
            type: 'success',
            duration: 1000
          })
        } else {
          this.$message({
            message: '修改打卡信息失败',
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
