<template>
  <div class="app-container">
    <div class="filter-container">
      <el-select v-model="listQuery.sort" style="width: 140px" class="filter-item">
        <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key" />
      </el-select>
      <el-input v-model="listQuery.title" placeholder="内容" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">搜索</el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-circle-plus-outline" @click="addDialog=true">添加</el-button>
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
      <el-table-column align="center" label="板块图片" width="120">
        <template slot-scope="{row}">
          <img style="width: 60px;height: 60px;border-radius: 10%" alt :src="row.imgUrl">
        </template>
      </el-table-column>
      <el-table-column align="center" label="板块名字">
        <template slot-scope="{row}">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="板块介绍">
        <template slot-scope="{row}">
          <span>{{ row.content }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="申请者id" width="80">
        <template slot-scope="{row}">
          <span>{{ row.userId }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="创建时间">
        <template slot-scope="{row}">
          <span>{{ row.date }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="板块状态" width="100">
        <template slot-scope="{row}">
          <span>{{ row.status+"("+(row.status==='1'?'已通过':'待审核')+")" }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button v-if="row.status==='0'" type="success" size="mini" @click="changePlateStatus(1,row.id)">
            通过
          </el-button>
          <el-button v-else type="info" size="mini" @click="changePlateStatus(0,row.id)">
            拒绝
          </el-button>
          <el-button type="primary" size="mini" @click="editPlate(row.name,row.imgUrl,row.content,row.id)">
            编辑
          </el-button>
          <el-button size="mini" type="danger" @click="deletePlate(row.id)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog
      title="输入板块信息"
      :visible.sync="addDialog"
      width="20%"
    >
      <el-form label-width="80px">
        <el-form-item label="板块名字"><el-input v-model="dialogs.name"/></el-form-item>
        <el-form-item label="板块图片"><el-input v-model="dialogs.imgUrl"/></el-form-item>
        <el-form-item label="板块内容"><el-input v-model="dialogs.content"/></el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialog = false">取 消</el-button>
        <el-button type="primary" @click="addPlate">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog
      title="修改板块信息"
      :visible.sync="editDialog"
      width="20%"
    >
      <el-form label-width="80px">
        <el-form-item label="板块名字"><el-input v-model="dialogs.name"/></el-form-item>
        <el-form-item label="板块图片"><el-input v-model="dialogs.imgUrl"/></el-form-item>
        <el-form-item label="板块内容"><el-input v-model="dialogs.content"/></el-form-item>
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
      addDialog: false,
      editDialog: false,
      dialogs: {
        name: '',
        imgUrl: '',
        content: '',
        plateID: ''
      },
      listQuery: {
        title: undefined,
        sort: 'id'
      },
      sortOptions: [
        { label: 'id', key: 'id' },
        { label: '板块名字', key: 'name' },
        { label: '申请者id', key: 'userId' },
        { label: '板块状态', key: 'status' }
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
      this.tools.requests(this.G.SERVER + '/api/admin/plate/get/plateList', {}, 'get').then((response) => {
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
            case 'name':
              if (this.allList[i].name.indexOf(this.listQuery.title) === -1) {
                continue
              }
              break
            case 'userId':
              if (this.allList[i].userId !== this.listQuery.title) {
                continue
              }
              break
            case 'status':
              if (this.allList[i].status !== this.listQuery.title) {
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
    // 添加新的板块
    addPlate() {
      // 上传代码
      this.tools.requests(this.G.SERVER + '/api/admin/plate/add', this.dialogs, 'post').then((response) => {
        if (response != null && response.code === 1) {
          this.$message.success('添加板块成功')
          this.addDialog = false
          this.getList()
        } else {
          this.$message.error('添加失败')
        }
      })
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
    // 删除板块
    deletePlate(id) {
      this.tools.requests(this.G.SERVER + '/api/admin/plate/delete', { plateID: id }, 'get').then((response) => {
        if (response != null && response.code === 1) {
          this.$message.success('删除板块成功')
          this.getList()
        } else {
          this.$message.error('删除板块失败')
        }
      })
    },
    // 打开编辑框
    editPlate(name, imgUrl, content, plateID) {
      // 板块内容赋值
      this.dialogs.name = name
      this.dialogs.imgUrl = imgUrl
      this.dialogs.content = content
      this.dialogs.plateID = plateID
      this.editDialog = true
    },
    // 修改板块信息
    editPlateInfo() {
      this.tools.requests(this.G.SERVER + '/api/admin/plate/change/plateInfo', this.dialogs, 'post').then((response) => {
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
