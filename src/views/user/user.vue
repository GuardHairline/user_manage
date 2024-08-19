<template>
  <div class="dashboard-container">
    <el-form ref="tableData" :model="tableData" label-width="80px" :inline="true" size="small">
      <el-form-item label="用户名称">
        <el-input v-model="tableData.name" placeholder="请输入用户名称" clearable="" />
      </el-form-item>
      <el-form-item label="创建时间">
        <el-date-picker v-model="tableData.minCreateTime" type="datetime" placeholder="起始时间" class="data-picker" />
        <el-date-picker v-model="tableData.maxCreateTime" type="datetime" placeholder="截止时间" class="data-picker" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="getUserList">查询</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <!-- 用户操作按钮 -->
    <div>
      <el-button type="primary" icon="el-icon-plus" size="mini" @click="handleCreateUser">新增</el-button>
      <el-button type="danger" icon="el-icon-delete" size="mini" @click="handleBatchDelete">删除</el-button>
      <el-button type="info" icon="el-icon-upload2" size="mini" @click="handleImportUser">导入</el-button>
    </div>
    <!-- 用户列表 -->

    <el-table :data="tableData.list" @selection-change="val => tableData.selection = val"
      @sort-change="handleSortChange" style="width: 100%">
      <el-table-column type="index" width="60" />
      <el-table-column type="selection" width="50" />
      <el-table-column width="60">
        <template slot-scope="scope">
          <img :id="'avatar-' + scope.row.id" class="table-avatar">
        </template>
      </el-table-column>
      <el-table-column prop="userName" label="用户名" sortable="custom" />
      <el-table-column prop="trueName" label="真实姓名" sortable="custom" />
      <el-table-column prop="roleList" label="角色" sortable="custom" />
      <el-table-column prop="createTime" label="创建时间" sortable="custom" />
      <el-table-column prop="status" label="是否激活" sortable="custom" width="100">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.status" :active-value="1" :inactive-value="0"
            @change="handleSwitch(scope.row)" />
        </template>
      </el-table-column>
      <el-table-column label="操作" width="180">
        <template slot-scope="scope">
          <el-button type="text" size="mini" icon="el-icon-edit" @click="handleEdit(scope.row)">编辑</el-button>
          <el-button type="text" size="mini" icon="el-icon-delete" @click="handleDelete(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <el-pagination @size-change="getUserList" @current-change="getUserList" :total="tableData.total"
      :current-page.sync="tableData.pageNum" :page-sizes="[10, 20, 30, 40]" :page-size.sync="tableData.pageSize"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>

    <!-- 用户编辑/创建窗口 -->
    <el-dialog class="user-edit-dialog" :title="userEditForm.id ? '用户编辑' : '新增用户'" :visible.sync="userEditDialogVisible"
      width="50%" top="8vh">
      <el-form ref="userEditForm" status-icon :model="userEditForm" label-width="80px"
        :rules="userEditForm.id ? userUpdateRules : userCreateRules">
        <el-form-item label="用户名" prop="userName">
          <el-input v-model="userEditForm.userName" />
        </el-form-item>
        <el-form-item label="真实姓名">
          <el-input v-model="userEditForm.trueName" />
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="userEditForm.password" />
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="userEditForm.email" />
        </el-form-item>
        <el-form-item label="性别">
          <el-radio-group v-model="userEditForm.gender">
            <el-radio :label="0">男</el-radio>
            <el-radio :label="1">女</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="地址">
          <el-input v-model="userEditForm.address" />
        </el-form-item>
        <el-form-item label="简介">
          <el-input v-model="userEditForm.introduction" />
        </el-form-item>
        <el-form-item label="电话">
          <el-input v-model="userEditForm.phone" />
        </el-form-item>
        <el-form-item label="角色" prop="roleIds">
          <el-select v-model="userEditForm.roleIds" multiple placeholder="请选择角色">
            <el-option v-for="role in allRoles" :key="role.id" :label="role.name" :value="role.id" />
          </el-select>
        </el-form-item>
        <el-form-item label="头像">
          <el-upload class="avatar-uploader" action="" :auto-upload="false" :show-file-list="false"
            :on-change="file => handleAvatarChange(file)">
            <!-- <img v-if="avatarUploadData.url" :src="avatarUploadData.url" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon" /> -->
          </el-upload>
          <!-- <el-button v-if="avatarUploadData.raw" size="mini" @click="resetUploadData(false)">重置</el-button> -->
        </el-form-item></el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="userEditDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import md5 from 'js-md5'
import * as UserApi from '@/api/user'
const copyObject = obj => JSON.parse(JSON.stringify(obj))

export default {
  name: 'User',
  data() {
    return {
      tableData: {
        name: '',
        minCreateTime: '',
        maxCreateTime: '',
        list: [{
          id: 1,
          userName: 'admin',
          trueName: '张三',
          roleList: ['管理员'],
          createTime: '2020-01-01 00:00:00',
          status: true
        }],
        selection: '',
        total: 1,
        pageNum: 1,
        pageSize: 10
      },
      userEditForm: {
        id: '',
        userName: '',
        trueName: '',
        password: '',
        email: '',
        gender: '',
        address: '',
        introduction: '',
        phone: '',
        roleIds: []
      },
      userCreateRules: {
        userName: [{ required: true, trigger: 'blur', validator: this.userNameValidator }],
        password: [{ required: true, trigger: 'change', validator: this.passwordValidator }],
        roleIds: [{ required: true, trigger: 'change', validator: this.roleValidator }]
      },
      userUpdateRules: {
        userName: [{ required: true, trigger: 'blur', validator: this.userNameValidator }],
        password: [{ trigger: 'change', validator: this.passwordValidator }],
        roleIds: [{ required: true, trigger: 'change', validator: this.roleValidator }]
      },
      allRoles: [],
      userEditDialogVisible: false

    }
  },
  methods: {

    /**
     * 获取用户列表
     */
    getUserList() {
      UserApi.getUsers(this.tableData).then(res => {
        this.tableData.list = res.data.data.content
        this.tableData.total = res.data.data.totalElements
        this.$nextTick(() => {
          this.tableData.list.forEach(row => {
            this.getAvatar(row.id, row)
          })
        })
      })
    },

    /**
     * 获取所有角色
     */
    getAllRoles() {

    },

    /**
     * 重置查询条件
     */
    resetQuery() {
      this.tableData.userName = ''
      this.tableData.minCreateTime = ''
      this.tableData.maxCreateTime = ''
    },

    // 编辑用户信息

    handleEdit(row) {
      for (const key in this.userEditForm) {
        this.userEditForm[key] = row[key]
      }
      this.userEditDialogVisible = true
    },
    /**
     * 删除用户
     */
    handleDelete(userIds) {
      this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 对接删除用户接口
      })
    },
    /**
     * 重置用户编辑表单
     */
    resetUserEditForm() {
      for (const key in this.userEditForm) {
        this.userEditForm[key] = ''
      }
      this.userEditForm.roleIds = []
    },
    /**
     * 切换用户状态
     * @param {object} row 行数据
     */
    handleSwitch(row) {
    },

    /**
     * 添加或更新用户
     */
    addOrUpdateUser() {
      this.$refs.userEditForm.validate(valid => {
        if (valid) {
          const params = copyObject(this.userEditForm)
          if (!params.password) {
            delete params.password
          } else {
            params.password = md5(params.password)
          }
        }
      })
    },

    /**
     * 创建用户
     */
    handleCreateUser() {
      this.userEditDialogVisible = true
      this.$nextTick(() => {
        this.$refs.userEditForm.clearValidate()
      })
    },

    /**
     * 批量删除用户
     */
    handleBatchDelete() {
      if (this.tableData.selection.length === 0) {
        this.$message.warning('请选择要删除的用户')
        return
      }
      const userIds = this.tableData.selection.map(item => item.id)
      this.handleDelete(userIds)
    },

    /**
     * 处理排序变化
     * @param {object} column 列对象
     * @param {string} prop 列属性
     * @param {string} order 排序方式
     */
    handleSortChange({ column, prop, order }) {
      this.tableData.orderBy = prop
      this.tableData.orderMethod = order === 'ascending' ? 'asc' : 'desc'
      this.getUserList()
    },

    /**
     * 处理头像变化
     * @param {object} file 文件对象
     */
    handleAvatarChange(file) {
      this.avatarUploadData.raw = file.raw
      this.avatarUploadData.url = URL.createObjectURL(file.raw)
    },

    /**
     * 重置上传数据
     * @param {boolean} clear 是否清空数据
     */
    resetUploadData(clear = true) {
      this.avatarUploadData.raw = null
      this.avatarUploadData.url = clear ? '' : this.avatarMap[this.userEditForm.id] || ''
    },

    /**
     * 打开用户编辑窗口
     */
    openUserEditForm() {
      this.userEditDialogVisible = true
      this.$nextTick(() => {
        this.$refs.userEditForm.clearValidate()
      })
    },

    /**
     * 打开用户导入窗口
     */
    handleImportUser() {
      this.$refs.userImportDialog.show(true)
    }
  }
}
</script>

<style scoped>
.data-picker {
  margin-right: 10px;
  width: 160px;
}
</style>
