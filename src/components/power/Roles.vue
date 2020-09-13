<template>
  <div>
    <!-- 面包屑导航栏 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
    <!-- 添加区 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDiaogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 权限列表渲染区域 -->
      <el-table :data="roleList" border stripe>
        <!-- 这个是展开列 -->
        <el-table-column type="expand">
            <!-- 通过嵌套循环进行渲染 -->
            <template slot-scope="scope">
                <el-row :class="['bdbottom', i1 === 0 ? 'bdtop':'', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                    <!-- 一级渲染 -->
                    <el-col :span="5">
                        <el-tag closable @close="removerightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                        <i class="el-icon-caret-right"></i>
                    </el-col>
                    <!-- 二级与三级渲染 -->
                    <el-col :span="18">
                    <!-- 通过for 循环 嵌套渲染二级 -->
                        <el-row :class="[i2 === 0 ? '':'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                            <el-col :span="6">
                                <el-tag type="success" closable @close="removerightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <el-col :span="18">
                                <!-- 最后的三级渲染 -->
                                <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" closable @close="removerightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                            </el-col>
                        </el-row>
                    </el-col>
                </el-row>
                <!-- <pre> -->
                    <!-- 格式化显示文本 -->
                    <!-- {{scope.row}} -->
                <!-- </pre> -->
            </template>
        </el-table-column>
        <!-- 普通索引列 -->
        <el-table-column type="index" label="id "></el-table-column>
        <el-table-column label="角色名" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <!-- 三个操作按钮 -->
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button type="primary" icon="el-icon-edit" size="mini">修改</el-button>
            <!-- 删除按钮 -->
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            <!-- 分配按钮 -->
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showsetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 打开的权限分配的对话框 -->
    <el-dialog title="权限分配详情框" :visible.sync="setRigthdialogVisible" width="50%" @close="setRigthdialogClose">
        <!-- 树形控件显示区 -->
        <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key='id' default-expand-all :default-checked-keys="defkeys" ref="treeRef"></el-tree>
        <span slot="footer" class="dialog-footer">
            <el-button @click="setRigthdialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色的列表数据
      roleList: [],
      // 控制分配权限框的显示与隐藏
      setRigthdialogVisible: false,
      // 所有权限数据的数据
      rightsList: [],
      // 树形控件的属性绑定对象
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中的节点的id值数组
      defkeys: [],
      // 当前即将分配权限的角色id
      roleId: ''
    }
  },
  created () {
    this.getrolesList()
  },
  methods: {
    // 获取所有列表数据
    async getrolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.roleList = res.data
      console.log(this.roleList)
    },
    // 根据id删除已有权限
    async removerightById (role, rightId) {
      // 弹框提示
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户取消删除则返回字符串 cancel
      // 如果用户确认删除则返回字符串 confirm
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      // 进行重新复制达到刷新列表保持当前数据为最新
      role.children = res.data
    },
    // 打开分配权限的对话框
    async showsetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('所有权限数据获取失败')
      }
      // 获取到的数据保存到data中
      this.rightsList = res.data
      console.log(this.rightsList)
      // 通过递归三级节点的id
      this.getLeafKeys(role, this.defkeys)
      this.setRigthdialogVisible = true
    },
    // 通过递归获取所有三级权限的id 并保存到 defkeys 数组中用来设置默认勾选框
    getLeafKeys (node, arr) {
    // 如果当前node节点不包含 children 则为三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      // 每循环一次拿到 item 根据这个再次调用这个函数
      node.children.forEach(item => this.getLeafKeys(item, arr)
      )
    },
    // 监听权限分配详情框的关闭事件
    setRigthdialogClose () {
      this.defkeys = []
    },
    // 点击为角色分配权限
    async allotRights () {
      const key = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedNodes()
      ]
      const idStr = key.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('权限分配失败')
      }
      this.$message.success('分配权限成功')
      this.getrolesList()
      this.setRigthdialogVisible = false
    }
  }
}
</script>

<style lang='less' scoped>
.el-tag{
    margin: 8px;
}

.bdtop{
    border-top: 1px solid #DDDDDD;
}

.bdbottom{
    border-bottom: 1px solid #DDDDDD;
}

.vcenter{
    display: flex;
    align-items: center;
}
</style>
