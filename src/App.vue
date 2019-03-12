<template>
  <div id="app">
      <el-header id="main-header"><h2>EH-Main</h2></el-header>
      <el-container id="main-container">
        <el-aside width="200px" class="aside">
          <el-menu
            router
            mode="vertical"
            :collapse="false">
            <el-submenu index="1" class="admin-links">
              <template slot="title">
                <i class="material-icons">build</i>
                <span slot="title">admin</span>
              </template>
              <el-menu-item index="/admin/home">
              <i class="material-icons">home</i>
              <span slot="title">admin home</span>
              </el-menu-item>
              <el-menu-item index="/admin/users">
                <i class="material-icons">verified_user</i>
                <span slot="title">users</span>
              </el-menu-item>
              <el-menu-item index="/admin/members">
                <i class="material-icons">person</i>
                <span slot="title">members</span>
              </el-menu-item>
              <el-menu-item index="/admin/events">
                <i class="material-icons">date_range</i>
                <span slot="title">events</span>
              </el-menu-item>
            </el-submenu>
            <el-menu-item index="/login">
              <i class="material-icons">lock</i>
              <span slot="title">login</span>
            </el-menu-item>
            <el-menu-item index="/">
              <i class="material-icons">home</i>
              <span slot="title">home</span>
            </el-menu-item>
          </el-menu>
        </el-aside>
        <el-main id="main-area">
          <router-view/>
        </el-main>
      </el-container>
  </div>
</template>

<script lang="ts">

export default {
  name: 'app',
  components: {
  },
  data() {
    return {
    };
  },
};
</script>

<style lang="scss">
$EH-black: #000;
$text-light: #EEE;

* {
  box-sizing: border-box;
}

body, html {
  margin: 0;
  min-height: 100vh;
}

#app {
  font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;

  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

/* ----- main ----- */

#main-header {
  background-color: $EH-black;
  color: $text-light;
}

#main-area {
  height: calc(100vh-60px);
  display: flex;
  padding: 20px;
  justify-content: center;
}

#main-container {
  flex-grow: 1;
}

.el-menu {
  height: 100%;
}

.aside {
  text-align: left;
}

.aside span {
  margin-left: 5px;
}

.admin-links {
  border-bottom: 1px solid #e6e6e6;
}

</style>
