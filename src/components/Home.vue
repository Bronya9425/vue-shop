<template>
  <el-container>
    <!-- 头部 -->
    <el-header>
      <div>
        <img src="../assets/logo.png" alt />
        <span>管理系统</span>
      </div>
      <el-button type="info" @click="logout">用户退出</el-button>
    </el-header>
    <!-- 页面 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside width="200px">
        <!-- 侧边栏菜单栏 -->
        <el-menu background-color="#b3c0d1" text-color="black" active-text-color="#00FFFF">
          <el-submenu :index="item.id" v-for="item in menulist" :key="item.id">
            <template slot="title">
              <i class="el-icon-menu"></i>
              <span>{{item.authName}}</span>
            </template>
            <!-- 次级菜单 -->
            <el-menu-item-group>
            <!-- 每组列表有符合的内容 -->
              <template slot="title" class="el-icon-document">
                <i class="el-icon-s-order"></i>
                {{item.children[0].authName}}
              </template>
              <el-menu-item index="1-1" class="el-icon-coin">{{item.authName.slice(0,2)}}{{item.id}}</el-menu-item>
              <el-menu-item index="1-2" class="el-icon-coin">{{item.authName.slice(0,2)}}{{item.id + 1}}</el-menu-item>
            </el-menu-item-group>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧内容 -->
      <el-main>Main</el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      // 左侧菜单栏数据
      menulist: [],
      index: []
    }
  },
  created () {
    this.getMenuList()
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.message.error(res.meta.msg)
      this.menulist = res.data
      console.log(res)
    }
  }
}
</script>

<style lang='less' scoped>
.el-container {
  height: 100%;
}
.el-header {
  background-color: #b3c0d1;
  color: #333;
  display: flex;
  justify-content: space-between;
  padding: 0;
  img {
    height: 48px;
  }
  > div {
    font-size: 28px;
    display: flex;
    align-items: center;
  }
}

.el-aside {
  background-color: #d3dce6;
}

.el-main {
  background-color: #e9eef3;
}
</style>
