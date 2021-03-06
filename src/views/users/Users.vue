<template>
  <div>
    <!-- 面包屑区域 -->
    <bread-crumb :content="content" />

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <!-- 搜索框区域 -->
          <el-input
            clearable
            @clear="getUserList"
            placeholder="请输入内容"
            v-model="queryInfo.query"
          >
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <!-- 添加用户功能 点击的时候弹出对话框 -->
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域  data是数据源 边框 鼠标覆盖高亮-->
      <el-table :data="userList" border stripe>
        <el-table-column type="index"></el-table-column>
        <!-- 渲染 姓名 邮箱 电话 -->
        <el-table-column
          v-for="item in columnList"
          :label="item.label"
          :prop="item.prop"
          :key="item.label"
        ></el-table-column>
        <el-table-column label="状态">
          <!-- 自定义插槽内容 作用域插槽 -->
          <template v-slot="scoped">
            <el-switch
              v-model="scoped.row.mg_state"
              @change="userStateChanged(scoped.row)"
            ></el-switch>
          </template>
        </el-table-column>

        <el-table-column label="操作" width="180px">
          <template v-slot="scoped">
            <!-- 修改按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scoped.row.id)"
            ></el-button>

            <!-- 删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeUserById(scoped.row.id)"
            ></el-button>

            <!-- 分配角色按钮 -->
            <el-tooltip content="分配角色" placement="top" :enterable="false">
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="mini"
                @click="setRole(scoped.row)"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页器 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10]"
        :page-size="queryInfo.pagesize"
        :total="total"
        layout="total, sizes, prev, pager, next, jumper"
      >
      </el-pagination>
      <!-- 添加用户对话框 -->
      <el-dialog
        title="添加用户"
        width="50%"
        :visible.sync="addDialogVisible"
        @close="addDialogClosed"
      >
        <!-- 内容主体区域 -->
        <el-form ref="addUserFormRef" label-width="70px" :model="addForm" :rules="addFormRules">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="电话" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>

        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUserInfo">添 加</el-button>
        </span>
      </el-dialog>

      <!-- 修改用户信息对话框 -->
      <el-dialog
        title="修改用户"
        width="50%"
        :visible.sync="editDialogVisible"
        @close="editDialogClosed"
      >
        <!-- 内容主体区域 -->
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
          <el-form-item label="用户名">
            <el-input v-model="editForm.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="电话" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUserInfo">修 改</el-button>
        </span>
      </el-dialog>

      <!-- 分配角色信息的对话框 -->
      <el-dialog
        title="分配角色"
        width="50%"
        :visible.sync="setRoleDialogVisible"
        @close="closedDialog"
      >
        <!-- 内容主体区域 -->
        <div>
          <p>当前的用户:{{ userInfo.username }}</p>
          <p>当前的角色:{{ userInfo.role_name }}</p>
          <p>
            分配新角色:
            <el-select v-model="selectedRoleId" placeholder="请选择">
              <el-option
                v-for="item in rolesList"
                :key="item.id"
                :label="item.roleName"
                :value="item.id"
              >
              </el-option>
            </el-select>
          </p>
        </div>

        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRoleDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="saveRoleInfo()">分 配</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //请求用户数据的参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      //面包屑数据
      content: [
        { content: '首页', path: { path: '/home' } },
        { content: '用户管理', path: '' },
        { content: '用户列表', path: '' }
      ],
      //渲染数据源
      columnList: [
        {
          label: '姓名',
          prop: 'username'
        },
        {
          label: '邮箱',
          prop: 'email'
        },
        {
          label: '电话',
          prop: 'mobile'
        },
        {
          label: '角色',
          prop: 'role_name'
        }
      ],
      //用户列表
      userList: [],
      // 总页数
      total: 0,
      // 控制添加用户对话框
      addDialogVisible: false,
      // 编辑添加用户对话框
      editDialogVisible: false,
      //添加用户的表单信息
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      //添加用户的表单验证规则
      addFormRules: {
        username: [
          {
            required: true,
            message: '请输入用户名',
            trigger: 'blur'
          },
          {
            min: 3,
            max: 10,
            message: '用户名的长度在3~10个字符之间',
            trigger: 'blur'
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          },
          {
            min: 6,
            max: 15,
            message: '密码的长度在6~15个字符之间',
            trigger: 'blur'
          }
        ],
        email: [
          {
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          },
          {
            validator: this.checkEmail,
            trigger: 'blur'
          }
        ],
        mobile: [
          {
            required: true,
            message: '请输入电话号码',
            trigger: 'blur'
          },
          {
            validator: this.checkMobile,
            trigger: 'blur'
          }
        ]
      },
      // 验证是否通过
      valid: false,
      // 修改用户验证是否通过
      editValid: false,
      // 编辑用户的数据表单
      editForm: {},
      //编辑用户的表单验证规则
      editFormRules: {
        email: [
          {
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          },
          {
            validator: this.checkEmail,
            trigger: 'blur'
          }
        ],
        mobile: [
          {
            required: true,
            message: '请输入电话号码',
            trigger: 'blur'
          },
          {
            validator: this.checkMobile,
            trigger: 'blur'
          }
        ]
      },
      // 控制分配角色对话框
      setRoleDialogVisible: false,

      userInfo: {},
      // 所有角色的数据列表
      rolesList: [],
      //已选择的角色id值
      selectedRoleId: ''
    }
  },
  // 请求数据
  created() {
    this.getUserList()
  },
  methods: {
    //自定义邮箱验证规则
    checkEmail(rule, value, callback) {
      const regEmail = /^(\w)+(\.\w+)*@(\w)+((\.\w+)+)$/
      if (regEmail.test(value)) {
        //合法的邮箱
        callback()
      } else {
        callback(new Error('邮箱格式不正确'))
      }
    },
    //自定义手机验证规则
    checkMobile(rule, value, callback) {
      const regMobile = /^[1][3,4,5,7,8][0-9]{9}$/
      if (regMobile.test(value)) {
        //合法的手机号
        callback()
      } else {
        callback(new Error('手机号格式不正确'))
      }
    },

    //获取用户数据
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })

      if (res.meta.status !== 200) {
        return this.$Message.error('获取用户数据失败！')
      }
      this.userList = res.data.users
      this.total = res.data.total
    },
    //监听switch开关的变化
    async userStateChanged(userInfo) {
      const { data: res } = await this.$http.put(`/users/${userInfo.id}/state/${userInfo.mg_state}`)
      if (res.meta.status !== 200) {
        this.$Message.error('更新用户数据失败')
        userInfo.mg_state = !userInfo.mg_state
      } else {
        this.$Message.success('更新用户数据成功')
      }
    },
    //处理当前页显示几条的数据
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    //处理当前页数的数据
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    //添加新用户信息
    async addUserInfo() {
      this.$refs.addUserFormRef.validate(valid => {
        this.valid = valid
      })
      //添加用户表单验证不通过直接退出
      if (!this.valid) return
      // 发送添加用户的请求
      const { data: res } = await this.$http.post('users', this.addForm)
      if (res.meta.status !== 201) return this.$Message.error('添加用户数据失败!')
      this.$Message.success('添加用户数据成功~')
      //关闭添加用户对话框
      this.addDialogVisible = false
      //刷新列表
      this.getUserList()
    },
    //关闭添加用户对话框重置表单
    addDialogClosed() {
      this.$refs.addUserFormRef.resetFields()
    },
    //关闭修改对话框重置表单
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    //显示修改用户信息对话框
    async showEditDialog(id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        this.$message.error('查询用户信息失败！')
      }
      this.editForm = res.data
    },

    async editUserInfo() {
      this.$refs.editFormRef.validate(valid => {
        //报存编辑用户表单验证通过的布尔值
        this.editValid = valid
      })
      // 验证不通过直接退出
      if (!this.editValid) return
      // 解构赋值
      const { id, mobile, email } = this.editForm
      const { data: res } = await this.$http.put('users/' + id, {
        email,
        mobile
      })
      if (res.meta.status !== 200) return this.$Message.error('修改用户数据失败')
      this.$Message.success('修改用户数据成功!')
      // 刷新用户数据
      this.getUserList()
      // 关闭对话框
      this.editDialogVisible = false
    },
    //删除用户信息
    async removeUserById(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      //如果删除了返回字符串 confirm 否则 cancel
      if (confirmResult !== 'confirm') return this.$Message.info('已取消删除')
      //发起删除用户信息的请求
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) return this.$Message.error('请求删除失败')
      this.$Message.success('删除成功')
      // 删除成功刷新列表
      this.getUserList()
    },

    async setRole(userInfo) {
      this.userInfo = userInfo
      // 在展示对话框之前，获取所有角色的列表
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$Message.error('获取角色列表失败')
      this.rolesList = res.data
      this.setRoleDialogVisible = true
    },
    //监听分配角色按钮点击
    async saveRoleInfo() {
      if (!this.selectedRoleId) return this.$Message.error('请选择要分配的角色')

      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, {
        rid: this.selectedRoleId
      })

      if (res.meta.status !== 200) return this.$Message.error('分配角色失败')

      this.$Message.success('分配角色成功')
      // 刷新用户列表
      this.getUserList()
      //关闭对话框
      this.setRoleDialogVisible = false
    },
    // 关闭对话框把表单重置
    closedDialog() {
      this.selectedRoleId = ''
      this.userInfo = {}
    }
  }
}
</script>

<style lang="less" scoped></style>
