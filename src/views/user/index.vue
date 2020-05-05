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
      <el-table-column label="id" align="center" width="80">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="用户头像" align="center" width="90">
        <template slot-scope="{row}">
          <el-avatar :size="60" :src="row.imgUrl" />
        </template>
      </el-table-column>
      <el-table-column label="微信名称" align="center">
        <template slot-scope="{row}">
          <span>{{ row.nickName }}</span>
        </template>
      </el-table-column>
      <el-table-column label="注册时间" align="center">
        <template slot-scope="{row}">
          <span>{{ row.registeredTime }}</span>
        </template>
      </el-table-column>
      <el-table-column label="性别" width="80" align="center">
        <template slot-scope="{row}">
          <span>{{ row.sex }}</span>
        </template>
      </el-table-column>
      <el-table-column label="用户名" align="center">
        <template slot-scope="{row}">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="学院" align="center">
        <template slot-scope="{row}">
          <span>{{ row.college }}</span>
        </template>
      </el-table-column>
      <el-table-column label="用户openid" align="center">
        <template slot-scope="{row}">
          <span>{{ row.openid }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-popconfirm
            confirm-button-text="确认"
            cancel-button-text="取消"
            icon="el-icon-info"
            icon-color="red"
            title="确认删除改用户么？"
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
    <el-dialog title="编辑用户信息" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :model="temp" label-position="left" label-width="70px" style="width: 400px; margin-left:50px;">
        <el-form-item label="名字" prop="type">
          <el-input v-model="temp.name" />
        </el-form-item>
        <el-form-item label="性别" prop="timestamp">
          <el-select v-model="temp.sex" class="filter-item" placeholder="性别">
            <el-option v-for="item in sexSelect" :key="item.key" :label="item.display_name" :value="item.display_name" />
          </el-select>
        </el-form-item>
        <el-form-item label="学院" prop="timestamp">
          <el-select v-model="temp.college" class="filter-item" placeholder="学院">
            <el-option v-for="item in collegeSelect" :key="item.key" :label="item.college" :value="item.college" />
          </el-select>
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

// eslint-disable-next-line no-unused-vars
const sexSelect = [
  { key: 'man', display_name: '男' },
  { key: 'girl', display_name: '女' },
  { key: 'secret', display_name: '保密' }
]

// eslint-disable-next-line no-unused-vars
const collegeSelect = [
  { key: 0, college: '保密' },
  { key: 1, college: '资源环境与安全工程学院' },
  { key: 2, college: '土木工程学院' },
  { key: 3, college: '机电工程学院' },
  { key: 4, college: '信息与电气工程学院' },
  { key: 5, college: '计算机科学与工程学院' },
  { key: 6, college: '化学化工学院' },
  { key: 7, college: '数学与计算科学学院' },
  { key: 8, college: '物理与电子科学学院' },
  { key: 9, college: '生命科学学院' },
  { key: 10, college: '建筑与艺术设计学院' },
  { key: 11, college: '人文学院' },
  { key: 12, college: '外国语学院' },
  { key: 13, college: '马克思主义学院' },
  { key: 14, college: '教育学院' },
  { key: 15, college: '商学院' },
  { key: 16, college: '艺术学院' },
  { key: 17, college: '体育学院' },
  { key: 18, college: '法学与公共管理学院' },
  { key: 19, college: '材料科学与工程学院' }
]

export default {
  name: 'ComplexTable',
  data() {
    return {
      tableKey: 0,
      list: null, // 存储用户信息
      pageList: null, // 存储当前页面的用户信息
      total: 0, // 用于分页的显示总数
      listLoading: true, // 是否显示加载状态
      sexSelect, // 性别选择
      collegeSelect, // 学院选择
      temp: {// 编辑的时候的板块
        id: -1,
        imgUrl: '',
        nickName: '',
        registeredTime: '',
        sex: '',
        name: '',
        college: '',
        openid: ''
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
      this.tools.requests(this.G.SERVER + '/api/admin/user/get/userList', {}, 'get').then((response) => { // 调用接口获取用户信息
        if (response != null && response.code === 1) {
          this.list = response.data
          this.pageList = response.data.slice(0, 10)
          this.total = response.data.length // 数据总数
        } else {
          this.$message({
            message: '获取用户信息失败',
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
    handleDeleteUser(row) { // 删除用户
      this.tools.requests(this.G.SERVER + '/api/admin/user/delete', {
        userID: row.id
      }, 'get').then((response) => { // 调用接口获取用户信息
        if (response != null && response.code === 1) {
          const index = this.list.findIndex(v => v.id === row.id) // 查找下标
          this.list.splice(index, 1) // 模拟修改数据
          this.total = this.list.length // 删除了话题后要修改这里
          this.pageList = this.list.slice((this.currentPage - 1) * 10, this.currentPage * 10)
          this.$message({
            message: '操作成功',
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
    handleUpdate(row) { // 修改用户信息
      console.log('每一行数据', row)
      this.temp = row
      console.log('temp', this.temp)
      this.temp.timestamp = new Date(this.temp.timestamp)
      this.dialogFormVisible = true // 开启弹框
    },
    updateData() { // 更新用户信息
      const temp = this.temp
      this.tools.requests(this.G.SERVER + '/api/user/update/userInfo', {
        name: temp.name,
        sex: temp.sex,
        college: temp.college,
        openid: temp.openid
      }, 'post').then((response) => { // 调用接口获取用户信息
        if (response != null && response.code === 1) {
          const index = this.list.findIndex(v => v.id === this.temp.id) // 查找下标
          this.list.splice(index, 1, this.temp) // 模拟修改数据
          this.$notify({
            title: '成功',
            message: '更新成功',
            type: 'success',
            duration: 1000
          })
        } else {
          this.$message({
            message: '修改用户信息失败',
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
