<template>
  <div class="users">
    <form class="form-app form-edit" @submit.prevent="update">
      <div class="alert alert-info" v-if="notice">{{ notice }}</div>
      <div class="alert alert-danger" v-if="error">{{ error }}</div>
      <div class="form-group">
        <label>Email address - {{ user.email }}</label>
      </div>
      <div class="form-group">
        <select v-model="user.role" class="form-control" id="role">
          <option value='admin'>Admin</option>
          <option value='manager'>Manager</option>
          <option value='user'>User</option>
        </select>
      </div>
      <button type="submit" class="btn btn-primary mb-3">Update</button>
      <div>
        <router-link to="/admin/users">Users</router-link>
      </div>
    </form>
  </div>
</template>

<script>

import { mapGetters } from 'vuex'

export default {
  name: 'UserEdit',
  computed: {
    ...mapGetters({
      isSignedIn: 'isSignedIn',
      isAdmin: 'isAdmin',
      currentUserId: 'currentUserId'
    })
  },
  data () {
    return {
      error: '',
      notice: '',
      user: {}
    }
  },
  created () {
    this.checkSignedIn()
  },
  methods: {
    update () {
      this.$http.secured.patch(`/admin/users/${this.$route.params.id}`, { user: { role: this.user.role } })
        .then(response => this.updateSuccessful(response))
        .catch(error => this.updateFailed(error))
    },
    updateSuccessful (response) {
      this.notice = 'User updated'
      this.error = ''
    },
    updateFailed (error) {
      this.error = (error.response && error.response.data && error.response.data.error) || ''
      this.notice = ''
    },
    setError (error, text) {
      this.error = (error.response && error.response.data && error.response.data.error) || text
      this.notice = ''
    },
    checkSignedIn () {
      this.$http.secured.get(`/admin/users/${this.$route.params.id}`)
        .then(response => {
          if (this.currentUserId === response.data.id) {
            this.$router.replace('/')
            return
          }
          this.user = response.data
        })
        .catch(error => { this.setError(error, 'Something went wrong') })
    }
  }
}
</script>
