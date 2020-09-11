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
      <el-aside :width="iscollapse ? '64px' : '200px'">
        <!-- 侧边栏内容 -->
        <div class="tog-button" @click="checkis">|||</div>
        <el-menu background-color="#b3c0d1" text-color="black" active-text-color="#00FFFF" unique-opened :collapse="iscollapse" router :default-active="actpath">
          <el-submenu :index="item.id" v-for="item in menulist" :key="item.id">
            <template slot="title">
              <i :class="iconset[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <!-- 次级菜单 -->
            <el-menu-item-group>
            <!-- 每组列表有符合的内容 -->
              <template slot="title" class="el-icon-document">
                <i :class="iconset[item.id]"></i>
                {{item.children[0].authName}}
              </template>
              <el-menu-item :index="'/' + item.path" class="el-icon-caret-right" @click="savestat('/' + item.path)">{{item.authName.slice(0,2)}}{{item.id}}</el-menu-item>
              <!-- <el-menu-item :index="'/' + item.path + 1" class="el-icon-caret-right">{{item.authName.slice(0,2)}}{{item.id + 1}}</el-menu-item> -->
            </el-menu-item-group>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧内容 -->
      <el-main>
      <!-- 路由占位符进行重定向到子页面 -->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      // 左侧菜单栏数据
      menulist: [],
      iconset: {
        125: 'el-icon-user-solid',
        103: 'el-icon-s-opportunity',
        101: 'el-icon-money',
        102: 'el-icon-notebook-2',
        145: 'el-icon-s-data'
      },
      // 是否折叠
      iscollapse: false,
      // 激活的地址保存
      actpath: ''
    }
  },
  // 生命周期函数
  created () {
    this.getMenuList()
    this.actpath = window.sessionStorage.setItem('actpath')// 得到保存的值
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
    },
    checkis () {
      // 通过取反值来设置菜单栏的展开与折叠
      this.iscollapse = !this.iscollapse
    },
    savestat (actpath) {
      // 保存链接的接受状态
      window.sessionStorage.setItem('actpath', actpath)
      this.actpath = actpath
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
  .el-menu{
    border-right: none
  }
}

.el-main {
  background-color: #e9eef3;
}

.tog-button {
  background-color: #646464;
  font-size: 10px;
  line-height: 25px;
  text-align: center;
  letter-spacing: 0.3em;
  cursor: pointer;
}
</style>
