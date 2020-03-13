<template>
  <el-container>
    <el-header v-bind:style="{ height: '80px', borderBottom: '1px solid #ced4da' }">
      <AppHeader></AppHeader>
    </el-header>
    <el-container>
      <el-container style="background-color: #eee">
        <el-main>
          <router-view></router-view>
        </el-main>
      </el-container>
      <!-- <el-aside>
        <AppSideBar></AppSideBar>
      </el-aside> -->
    </el-container>
  </el-container>
</template>

<script>

import { mapGetters } from 'vuex'
import AppHeader from '@/components/AppHeader'
import AppSideBar from '@/components/AppSideBar'

export default {
  name: 'App',
  components: {
    AppHeader,
    AppSideBar
  },
  computed: {
    ...mapGetters({
      isAdmin: 'isAdmin',
      isManager: 'isManager'
    })
  },
  methods: {
    showSideBarMenu () {
      return this.isAdmin || this.isManager
    }
  }
}
</script>

<style>
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
