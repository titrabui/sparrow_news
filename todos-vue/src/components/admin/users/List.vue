<template lang="pug">
  el-row
    el-card.box-card(:body-style="{ padding: '0px' }")
      el-row.box-card-header
        span.box-header-title
          i(class="el-icon-user")
          span  Users Management
      el-row.user-search-input
        el-input(
          placeholder="Search users"
          prefix-icon="el-icon-search"
          v-model="search"
          style="width: 400px"
        )
      el-row(style="padding: 14px 20px")
        el-table(:data="users.filter(data => !search || data.email.toLowerCase().includes(search.toLowerCase())).slice((currentPage - 1) * pageSize, (currentPage - 1) * pageSize + pageSize)" style="width: 100%" empty-text="No data")
          el-table-column(label="User ID" prop="id" width="80")
          el-table-column(label="Avatar" width="80" align="center" v-slot="{ row }")
            el-popover(
              placement="top-start"
              width="200"
              trigger="hover"
              :disabled="row.avatar.url === null"
              popper-class="avatar-popover")
              el-avatar(:src="row.avatar.url" :size="200" shape="square")
              el-avatar(v-if="row.avatar.url" slot="reference" :src="row.avatar.url" :size="40" shape="square")
              el-avatar(v-else slot="reference" :size="40" shape="square") {{ row.email.charAt(0).toUpperCase() }}
          el-table-column(label="Status" width="80" align="center" v-slot="{ row }")
            el-tooltip(:content="row.content_visibility ? 'Active' : 'Deactive'")
              el-switch(v-model="row.content_visibility" active-color="#13ce66" inactive-color="#ff4949")
          el-table-column(label="Email" prop="email" min-width="200")
          el-table-column(label="Display name" min-width="200" v-slot="{ row }")
            span {{ row.display_name || '-' }}
          el-table-column(label="Role" prop="role" min-width="100" v-slot="{ row }")
            el-select(v-if="isAdmin" v-model="row.role" @change="changeUserRole(row.id, $event)" @focus="backupRole($event)")
              el-option(
                v-for="item in userRoles"
                :key="item"
                :label="item"
                :value="item")
            span(v-if="isManager") {{ row.role }}
          el-table-column(label="Posts" align="right" min-width="100" v-slot="{ row }")
            span {{ row.posts }}
          el-table-column(label="Comments" align="right" min-width="100" v-slot="{ row }")
            span {{ row.comments }}
          el-table-column(label="Birthday" min-width="150" align="right" v-slot="{ row }")
            span {{ (row.birthday && birthday(row.birthday)) || '-' }}
          el-table-column(label="About" min-width="200" v-slot="{ row }")
            span {{ row.about || '-' }}
          el-table-column(label="Registered at" min-width="150" align="right" v-slot="{ row }")
            span {{ createdAt(row.created_at) }}
          el-table-column(fixed="right" label="Operations" width="120" :align="isManager ? 'center' : 'left'" v-slot="{ row }")
            el-tooltip(content="View News")
              el-button(@click="viewUserPost(row.id)" type="success" icon="el-icon-news" circle)
            el-tooltip(content="Delete")
              el-button(v-if="canDeleteUser(row.id)" @click="removeUser(row.id)" type="danger" icon="el-icon-delete" circle)
      el-row(style="padding: 14px 20px; text-align: center;")
        el-pagination(
          background
          layout="prev, pager, next"
          :current-page.sync="currentPage"
          :page-size="pageSize"
          :pager-count="5"
          :total="users.length"
        )
</template>

<script>

import moment from 'moment'
import { mapGetters, mapActions } from 'vuex'
import { store } from '@/store'

export default {
  name: 'UsersList',
  computed: {
    ...mapGetters({
      isAdmin: 'isAdmin',
      isManager: 'isManager',
      currentUserId: 'currentUserId',
      users: 'users/users'
    })
  },
  data () {
    return {
      search: '',
      userRoles: ['admin', 'manager', 'user'],
      roleBackupValue: '',
      currentPage: 1,
      pageSize: 10
    }
  },
  created () {
    this.fetchUsers()
  },
  methods: {
    ...mapActions({
      fetchUsers: 'users/fetchUsers',
      deleteUser: 'users/deleteUser'
    }),
    setError (error, text) {
      this.$notify({
        type: 'error',
        title: text,
        message: error.response && error.response.data && error.response.data.error
      })
    },
    createdAt (createdAt) {
      return moment(createdAt).format('DD - MM - YYYY, H:mm:ss')
    },
    birthday (birthday) {
      return moment(birthday).format('DD - MM - YYYY')
    },
    backHome () {
      this.$router.replace('/')
    },
    viewUserPost (userId) {
      store.commit('setPostManageUserId', { userId: userId })
      this.$router.replace('/news-manage')
    },
    removeUser (userId) {
      this.$confirm('This will permanently delete the user. Continue?', 'Warning', {
        confirmButtonText: 'OK',
        cancelButtonText: 'Cancel',
        type: 'warning'
      }).then(() => {
        this.deleteUser(userId)
      }).catch(() => {})
    },
    canDeleteUser (userId) {
      return this.isAdmin && this.currentUserId !== userId
    },
    changeUserRole (userId, role) {
      this.$confirm('This will change role of the user. Continue?', 'Warning', {
        confirmButtonText: 'OK',
        cancelButtonText: 'Cancel',
        type: 'warning'
      }).then(() => {
        this.$http.secured.patch(`admin/users/${userId}`, { role: role })
          .then(response => {
            this.$notify({
              type: 'success',
              message: 'Change completed'
            })
          })
          .catch(error => {
            this.fetchUsers()
            this.setError(error, 'Cannot update user')
          })
      }).catch(() => {})
    },
    backupRole ($event) {
      this.roleBackupValue = $event.target.value
    }
  }
}
</script>

<style lang="css">
  .user-search-input {
    padding: 10px 20px;
    text-align: right;
  }

  .avatar-popover {
    padding: 0px !important;
  }
</style>
