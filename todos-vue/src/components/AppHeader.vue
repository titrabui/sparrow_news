<template lang="pug">
  nav.navbar.rounded(style="padding: 0.2rem 1rem")
    ul.navbar(style="padding: 0")
      li.nav-item.home
        router-link.nav-link(to="/")
          img(src="@/assets/images/logo.png" height="70")
    ul.navbar
      li.nav-item
        el-input(
          placeholder="Search news"
          prefix-icon="el-icon-search"
          v-model="newsSearch"
          style="width: 400px"
          size="medium")
    ul.navbar.ml-auto
      li.nav-item
        div(v-if="isSignedIn")
          el-tooltip(content="Create news" placement="bottom")
            el-button(type="success" icon="el-icon-edit" @click="redirectToPage('news-create')" size="medium" round style="font-weight: bold") Create news
          el-dropdown(@command="handleCommand" style="margin-left: 10px; vertical-align: middle;")
            el-button.dropdown-signedin(round)
              span.dropdown-info
                span.signin-info
                  el-avatar(v-if="currentUser.avatar.url" :src="currentUser.avatar.url" shape="square" :size="30")
                  el-avatar(v-else shape="square" :size="30") {{ (currentUser.display_name || currentUser.email).charAt(0).toUpperCase() }}
                  span.user-info
                    span.user-account {{ currentUser.display_name || currentUser.email }}
                    span.user-role {{ currentUser.role }}
            el-dropdown-menu(slot="dropdown")
              el-dropdown-item(command="news-manage" icon="el-icon-platform-eleme") News Management
              el-dropdown-item(v-if="isAdmin || isManager" command="admin/users" icon="el-icon-user-solid") Users Management
              el-dropdown-item(command="settings" icon="el-icon-s-tools") Settings
              el-dropdown-item(command="signout" icon="el-icon-back") Log Out
        div(v-else)
          el-button(@click="openDialog('signin')" size="medium" round style="font-weight: bold") LOG IN
          el-button(@click="openDialog('signup')" size="medium" round type="primary" style="font-weight: bold") SIGN UP
    SignForm(ref="signForm" :sign-type="signFormType")
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import SignForm from './SignForm'

export default {
  name: 'AppHeader',
  components: {
    SignForm
  },
  computed: {
    ...mapGetters({
      isSignedIn: 'isSignedIn',
      isAdmin: 'isAdmin',
      isManager: 'isManager',
      currentUser: 'currentUser'
    })
  },
  data () {
    return {
      newsSearch: '',
      signFormType: 'in'
    }
  },
  methods: {
    ...mapActions({
      unsetCurrentUser: 'app/unsetCurrentUser'
    }),
    setError (error, text) {
      this.error = (error.response && error.response.data && error.response.data.error) || text
    },
    signOut () {
      this.$http.secured.delete('/signin')
        .then(response => {
          this.unsetCurrentUser()
          this.$router.replace('/')
        })
        .catch(error => this.setError(error, 'Cannot sign out'))
    },
    showAdminLink () {
      return this.isAdmin || this.isManager
    },
    redirectToPage (route) {
      this.$router.replace(`/${route}`)
    },
    openDialog (ref) {
      if (['signin', 'signup'].includes(ref)) {
        this.signFormType = ref.replace('sign', '')
        this.$refs.signForm.dialogVisible = true
      } else {
        this.redirectToPage(ref)
      }
    },
    handleCommand (command) {
      if (command === 'signout') {
        this.signOut()
      } else {
        this.openDialog(command)
      }
    }
  }
}
</script>

<style lang="css" scoped>
  ul {
    list-style-type: none;
  }

  .nav-bar {
    padding: .5rem 0;
  }

  .nav-link {
    padding: 0 1rem;
  }

  .home {
    width: 200px;
  }

  .dropdown-info {
    display: flex;
    align-items: center;
  }

  .signin-info {
    display: flex;
    text-align: left;
    align-items: center;
  }

  .user-info {
    display: block;
    margin-left: 5px
  }

  .user-account {
    font-size: 12px;
    font-weight: 500;
    line-height: 16px;
    color: var(--newCommunityTheme-bodyText);
    display: block;
    white-space: nowrap;
  }

  .user-role {
    font-size: 11px;
    font-weight: 500;
    line-height: 16px;
    color: #a8aaab;
    text-transform: capitalize;
  }

  .dropdown-signedin {
    padding: 3px 10px;
  }
</style>
