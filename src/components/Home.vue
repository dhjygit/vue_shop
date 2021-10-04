<template>
  <el-container class="home-container">
    <el-header>
      <div class="header-left">
        <img src="../assets/heima.png" alt="logo">
        <span>电商后台管理系统</span>
      </div>
      <div class="header-right">
        <el-button type="info" @click="logout">退出登陆</el-button>
      </div>
    </el-header>
    <el-container>
      <el-aside :width="isCollapse?'46px':'180px'">
        <div class="aside-toggle" @click="toggleCollapse">|||</div>
        <el-menu
          :default-active="navState"
          class="el-menu-vertical-demo"
          background-color="#333744"
          text-color="white"
          active-text-color="#409EFF"
          unique-opened
          router
          :collapse="isCollapse"
          :collapse-transition="false">
          <el-submenu :index="''+menuItem.id" v-for="menuItem in menuList" :key="menuItem.id">
            <template slot="title">
              <i :class="iconsObj[menuItem.id]"></i>
              <span>{{ menuItem.authName }}</span>
            </template>
            <el-menu-item :index="'/'+subMenuItem.path" v-for="subMenuItem in menuItem.children" :key="subMenuItem.id"
                          @click="saveNavState('/'+subMenuItem.path)">
              <i class="el-icon-menu"></i>
              <span>{{ subMenuItem.authName }}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>
<script>
export default {
  name: 'Home',
  data () {
    return {
      menuList: [],
      iconsObj: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollapse: false,
      navState: ''
    }
  },
  created () {
    this.getMenuList()
    this.navState = window.sessionStorage.getItem('navState')
  },
  methods: {
    logout () {
      window.sessionStorage.removeItem('token')
      this.$router.push('/login')
      this.$message.success('退出成功')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
    },
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    saveNavState (navState) {
      window.sessionStorage.setItem('navState', navState)
      this.navState = navState
    }
  }
}
</script>
<style lang="scss" scoped>
.home-container {
  height: 100%;

  .el-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #373d41;
    padding-left: 0;
    color: white;

    .header-left {
      display: flex;
      align-items: center;

      span {
        margin-left: 15px;
      }
    }
  }

  .el-aside {
    background-color: #333744;
    transition: width .5s ease;
    overflow: hidden;

    .aside-toggle {
      color: white;
      text-align: center;
      font-size: 12px;
      line-height: 24px;
      letter-spacing: 1px;
      background-color: #4a5064;
      cursor: pointer;
    }

    .el-menu {
      border-right: 0px;

      /deep/ .el-submenu__title {
        padding: 0 15px !important;
      }
    }
  }

  .el-main {
    background-color: #eaedf1;
  }
}

.iconfont {
  margin-right: 15px;
}
</style>
