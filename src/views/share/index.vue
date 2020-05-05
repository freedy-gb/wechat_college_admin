<template>
  <div class="app-container">
    <div class="filter-container">
      <el-select v-model="listQuery.sort" style="width: 140px" class="filter-item">
        <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key" />
      </el-select>
      <el-input v-model="listQuery.title" placeholder="内容" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">搜索</el-button>
    </div>

    <el-table
      v-loading="listLoading"
      :data="list"
      style="width: 100%"
      border
      highlight-current-row
    >
      <el-table-column align="center" fixed label="id" width="80">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="图片" width="120">
        <template slot-scope="{row}">
          <img style="width: 60px;height: 60px;border-radius: 10%" alt :src="row.img">
        </template>
      </el-table-column>
      <el-table-column align="center" label="标题">
        <template slot-scope="{row}">
          <span>{{ row.title }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="所属板块">
        <template slot-scope="{row}">
          <span>{{ row.plate }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="作者id" width="80">
        <template slot-scope="{row}">
          <span>{{ row.userID }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="发布时间">
        <template slot-scope="{row}">
          <span>{{ row.time }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="点赞数" width="100">
        <template slot-scope="{row}">
          <span>{{ row.total }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="浏览量" width="100">
        <template slot-scope="{row}">
          <span>{{ row.view }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="评论数" width="100">
        <template slot-scope="{row}">
          <span>{{ row.comments }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="editPlate(row.title,row.rowImg,row.content,row.id,row.userID)">
            编辑
          </el-button>
          <el-button size="mini" type="danger" @click="deletePlate(row.id, row.userID)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog
      title="修改板块信息"
      :visible.sync="editDialog"
      width="30%"
    >
      <el-form label-width="80px">
        <el-form-item label="经验标题"><el-input v-model="dialogs.title" /></el-form-item>
        <el-form-item label="经验图片"><el-input v-model="dialogs.imgUrl" /></el-form-item>
        <el-form-item label="经验内容"><el-input :rows="5" v-model="dialogs.content" type="textarea" /></el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialog = false">取 消</el-button>
        <el-button type="primary" @click="editPlateInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import waves from '@/directive/waves' // waves directive
import Vue from 'vue'

export default {
  name: 'ComplexTable',
  directives: { waves },
  data() {
    return {
      list: null,
      allList: null,
      editDialog: false,
      dialogs: {
        title: '',
        shareID: '',
        userID: '',
        content: '',
        imgUrl: ''
      },
      listQuery: {
        title: undefined,
        sort: 'id'
      },
      sortOptions: [
        { label: 'id', key: 'id' },
        { label: '标题', key: 'title' },
        { label: '作者id', key: 'userId' },
        { label: '所属板块', key: 'plate' }
      ]
    }
  },
  created() {
    this.getList()
  },
  methods: {
    // 获取所有的板块
    getList() {
      this.listLoading = true
      // 获取所有的板块
      this.tools.requests(this.G.SERVER + '/api/admin/plate/get/shareList', {}, 'get').then((response) => {
        if (response != null && response.code === 1) {
          this.list = response.data
          this.allList = response.data
          this.listLoading = false
        }
      })
    },
    // 搜索结果
    handleFilter() {
      var list = []
      for (let i = 0; i < this.allList.length; i++) {
        if (this.listQuery.title !== undefined && this.listQuery.title !== '') {
          switch (this.listQuery.sort) {
            case 'id':
              if (this.allList[i].id !== this.listQuery.title) {
                continue
              }
              break
            case 'title':
              if (this.allList[i].title.indexOf(this.listQuery.title) === -1) {
                continue
              }
              break
            case 'userId':
              if (this.allList[i].userID !== this.listQuery.title) {
                continue
              }
              break
            case 'plate':
              if (this.allList[i].plate.indexOf(this.listQuery.title) === -1) {
                continue
              }
              break
            default:
              break
          }
        }
        list.push(this.allList[i])
      }
      Vue.set(this, 'list', list)
    },
    // 通过拒绝板块
    changePlateStatus(status, id) {
      this.tools.requests(this.G.SERVER + '/api/admin/plate/change/plateStatus', { status: status, plateID: id }, 'post').then((response) => {
        if (response != null && response.code === 1) {
          this.$message.success('修改状态成功')
          this.getList()
        } else {
          this.$message.error('修改状态失败')
        }
      })
    },
    // 删除经验
    deletePlate(id, userId) {
      this.tools.requests(this.G.SERVER + '/api/plate/delete/shareContent', { shareID: id, userID: userId }, 'post').then((response) => {
        if (response != null && response.code === 1) {
          this.$message.success('删除经验成功')
          this.getList()
        } else {
          this.$message.error('删除经验失败')
        }
      })
    },
    // 打开编辑框
    editPlate(name, img, content, shareID, userID) {
      // 板块内容赋值
      this.dialogs.title = name
      this.dialogs.imgUrl = img
      this.dialogs.content = content
      this.dialogs.userID = userID
      this.dialogs.shareID = shareID
      this.editDialog = true
    },
    // 修改板块信息
    editPlateInfo() {
      this.tools.requests(this.G.SERVER + '/api/plate/edit/shareContent', this.dialogs, 'post').then((response) => {
        if (response != null && response.code === 1) {
          this.$message.success('修改板块成功')
          this.editDialog = false
          this.getList()
        } else {
          this.$message.error('修改板块失败')
        }
      })
    }
  }
}
</script>
