<template>
  <el-container>
    <el-aside width="200px"><AppAside /></el-aside>
    <el-container>
      <el-header>
        <AppHeader />
      </el-header>
      <el-main>
        main
      </el-main>
    </el-container>
  </el-container>
</template>
<script>
import AppAside from './component/AppAside'
import AppHeader from './component/AppHeader'
export default {
  name: 'AooLayout',
  components: {
    AppAside,
    AppHeader
  },
  data() {
    return {}
  }
}
</script>
<style lang="less" scoped>
  .el-container {
    height: 100%;
  }
  .el-aside {
    background-color: #343B47;
  }
  .el-header {
    background-color: #A6BED0;
  }
  .el-main {
    background-color: #E5EFF5;
  }
</style>
