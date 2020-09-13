<template>
  <div>
    <!-- 面包屑导航栏 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索与添加区 -->
      <el-row :gutter="20">
        <!-- 搜索区 -->
        <el-col :span="6">
          <el-input placeholder="请输入内容" v-model="queryInfo.query " clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <!-- 添加区 -->
        <el-col :span="6">
          <el-button type="primary" @click="addDiaogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="userlist" border>
        <el-table-column type="index" label="id "></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="手机" prop="mobile"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <!-- 状态按钮 -->
        <el-table-column label="状态">
          <template slot-scope="scop">
            <el-switch v-model="scop.row.mg_state" @change="userStateChanged(scop.row.id)"></el-switch>
          </template>
        </el-table-column>
        <!-- 三个操作按钮 -->
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
            <!-- 删除按钮 -->
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
            <!-- 分配按钮 -->
            <el-tooltip class="item" effect="dark" content="进行分配" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 添加用户的对话框 -->
      <el-dialog title="添加用户" :visible.sync="addDiaogVisible" width="50%" @close="addDiaogClose">
        <!-- 内容区 -->
        <el-form :model="addFrom" :rules="addFromrules" ref="addFormRef" label-width="70px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addFrom.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addFrom.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addFrom.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="addFrom.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区 -->
        <span slot="footer">
          <el-button @click="addDiaogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 修改用户的对话框 -->
      <el-dialog title="用户修改" :visible.sync="editDiaogVisible" width="50%" @close="editDiaogClose">
        <el-form :model="editFrom" :rules="editFromRules" ref="editFromRef" label-width="70px">
          <el-form-item label="用户名">
            <el-input v-model="editFrom.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="editFrom.mobile"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editFrom.email"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer">
        <el-button @click="editDiaogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 分配角色的对话框 -->
      <el-dialog title="用户角色分配" :visible.sync="setdialogVisible" width="50%" @close=setroleDialogClose>
        <div>
          <p>当前的用户:{{userInfo.username}}</p>
          <p>当前的角色:{{userInfo.role_name}}</p>
          <p>分配新角色
            <el-select v-model="selectRolesId" placeholder="请选择">
              <el-option v-for="item in roleList" :key="item.id" :label="item.roleName" :value="item.id"></el-option>
            </el-select>
          </p>
        </div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setdialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 分页效果实现 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 4, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="toatal"
      ></el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    var checkEmail = (rule, value, cb) => {
      // 验证邮箱的正则表达式
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        // 合法邮箱
        return cb()
      }
      cb(new Error('邮箱不合法'))
    }
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^1(3|4|5|6|7|8|9)\d{9}$/
      if (regMobile.test(value)) {
        // 合法手机号
        return cb()
      }
      cb(new Error('手机号不合法'))
    }
    return {
      // 获取用户列表的参数对象
      queryInfo: {
        query: '',
        // 当前的页码
        pagenum: 1,
        // 当前每页显示几条数据
        pagesize: 4
      },
      userlist: [],
      // 获取到了几条数据
      toatal: 0,
      // 控制添加对话框的显示和隐藏
      addDiaogVisible: false,
      // 添加用户的表单数据
      addFrom: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加表单的验证规则
      addFromrules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名长度在3到10之间', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '密码不能为空', trigger: 'blur' },
          { min: 5, max: 10, message: '用户名长度在5到10之间', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '邮箱不能为空', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '手机不能为空', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 控制修改对话框的显示和隐藏
      editDiaogVisible: false,
      // 查到的用户信息
      editFrom: {},
      // 修改表单  的验证对象
      editFromRules: {
        email: [
          { required: true, message: '邮箱不能为空', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '手机不能为空', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 控制分配角色对话框的显示与隐藏
      setdialogVisible: false,
      // 需要被分配的用户信息
      userInfo: {},
      // 拿到的所有角色的数据列表
      roleList: [],
      // 以选中将要分配的角色id值
      selectRolesId: ''
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    // 获取用户列表数据
    async getUserList () {
      const { data: res } = await this.$http.get('users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) return this.$message.error('获取失败')
      this.toatal = res.data.total
      this.userlist = res.data.users
      console.log(this.userlist)
    },
    // 监听pagesize改变的事件
    handleSizeChange (newsize) {
      // console.log(newsize)
      this.queryInfo.pagesize = newsize
      this.getUserList()
    },
    // 监听页码值改变的事件
    handleCurrentChange (newpage) {
      // console.log(newpage)
      this.queryInfo.pagenum = newpage
      this.getUserList()
    },
    // 监听 switch 开关状态的改变
    async userStateChanged (userinfo) {
      console.log(userinfo)
      const { data: res } = await this.$http.put(
        `users/${userinfo.id}/state/${userinfo.mg_state}`
      )
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('修改失败')
      }
      this.$message.success('修改成功')
    },
    // 监听添加用户对话框的关闭
    addDiaogClose () {
      // 通过调用内置方法进行重置表单
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮添加新用户
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        // 若验证通过则发起添加请求
        const { data: res } = await this.$http.post('users', this.addFrom)
        if (res.meta.status !== 201) return this.$message.error('添加失败')
        this.$message.success('添加成功')
        // 隐藏添加用户的对话框
        this.addDiaogVisible = false
        // 重新获取用户列表数据
        this.getUserList()
      })
    },
    // 打开修改用户的对话框
    async showEditDialog (id) {
      // console.log(id)
      this.editDiaogVisible = true
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('未能找到此用户')
      this.editFrom = res.data
      console.log(res)
    },
    // 监听修改用户对话框的关闭
    editDiaogClose () {
      // 通过调用内置方法进行重置表单
      this.$refs.editFromRef.resetFields()
    },
    // 点击按钮修改用户数据并提交
    editUser () {
      this.$refs.editFromRef.validate(async valid => {
        if (!valid) return
        // 若验证通过则发起修改请求
        const { data: res } = await this.$http.put('users/' + this.editFrom.id, {
          email: this.editFrom.email,
          mobile: this.editFrom.mobile
        })
        if (res.meta.status !== 200) return this.$message.error('修改失败')
        // 隐藏添加用户的对话框
        this.editDiaogVisible = false
        // 重新获取用户列表数据
        this.getUserList()
        this.$message.success('修改成功')
      })
    },
    // 通过id删除对应的用户数据
    async removeUserById (id) {
      // 弹框消息提示
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户取消删除则返回字符串 cancel
      // 如果用户确认删除则返回字符串 confirm
      console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除操作')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      // 刷新用户列表数据
      this.getUserList()
      this.$message.success('删除成功')
    },
    // 分配角色的对话框
    async setRole (userdata) {
      this.userInfo = userdata
      // 展示对话框之前获取所有角色的列表
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取失败')
      this.roleList = res.data
      // console.log(this.roleList)
      this.setdialogVisible = true
    },
    // 点击确认按钮分配角色
    async saveRoleInfo () {
      if (!this.selectRolesId) return this.$message.error('请选择要分配的角色')
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectRolesId })
      if (res.meta.status !== 200) return this.$message.error('分配失败')
      this.$message.success('更新角色成功')
      // 刷新列表数据
      this.getUserList()
      // 隐藏对话框
      this.setdialogVisible = false
    },
    setroleDialogClose () {
      this.userInfo = ''
      this.selectRolesId = ''
    }
  }
}
</script>

<style lang='less' scoped>
</style>
