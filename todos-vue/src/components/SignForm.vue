<template lang="pug">
  el-dialog(:visible.sync="dialogVisible" width="60%")
    el-row.signin-container(:gutter="40")
      el-col.signin-form(:span="12")
        .signin-dialog-title {{ `Sign ${signType}` }}
        el-alert(
          v-if="error"
          :title="error"
          type="error"
          effect="dark"
          center
          :closable=false
        )
        el-form(label-position="top" label-width="100px")
          el-form-item
            template(slot="label")
              .signin-item-label Email address
            el-input(placeholder="Please input email" v-model="email")
          el-form-item
            template(slot="label")
              .signin-item-label Password
            el-input(placeholder="Please input password" v-model="password" show-password)
          el-form-item(v-if="signType === 'up'")
            template(slot="label")
              .signin-item-label Confirm Password
            el-input(placeholder="Please input confirm password" v-model="password_confirmation" show-password)
          el-form-item
            el-button(type="primary" style="width: 100%" round @click="signType === 'in' ? signin() : signup()" size="large") {{ `Sign ${signType}` }}
      el-col(:span="12" align="middle")
        el-form(size="large")
          el-form-item
            el-button.btn-signin.btn-google(round @click="signWithGoogle()")
              font-awesome-icon(:icon="['fab', 'google']")
              el-divider(direction="vertical")
              span {{ `Sign ${signType} with Google` }}
          el-form-item
            el-button.btn-signin.btn-facebook(round)
              font-awesome-icon(:icon="['fab', 'facebook-f']")
              el-divider(direction="vertical")
              span {{ `Sign ${signType} with Facebook` }}
          el-form-item
            el-button.btn-signin.btn-twitter(round)
              font-awesome-icon(:icon="['fab', 'twitter']")
              el-divider(direction="vertical")
              span {{ `Sign ${signType} with Twitter` }}
</template>

<script>

import { mapGetters, mapActions } from 'vuex'

export default {
  name: 'SignForm',
  props: {
    signType: {
      type: String,
      default: () => 'in'
    }
  },
  computed: {
    ...mapGetters({
      isSignedIn: 'isSignedIn'
    })
  },
  data () {
    return {
      email: '',
      password: '',
      password_confirmation: '',
      error: '',
      dialogVisible: false
    }
  },
  created () {
    this.checkSignedIn()
  },
  updated () {
    this.checkSignedIn()
  },
  methods: {
    ...mapActions({
      setCurrentUser: 'app/setCurrentUser',
      unsetCurrentUser: 'app/unsetCurrentUser'
    }),
    signin () {
      this.$http.plain.post('/signin', { email: this.email, password: this.password })
        .then(response => this.signSuccessful(response))
        .catch(error => this.signFailed(error))
    },
    signup () {
      this.$http.plain.post('/signup', { email: this.email, password: this.password, password_confirmation: this.password_confirmation })
        .then(response => this.signSuccessful(response))
        .catch(error => this.signFailed(error))
    },
    signSuccessful (response) {
      if (!response.data.csrf) {
        this.signFailed(response)
        return
      }

      this.dialogVisible = false
      this.$http.plain.get('/me').then(meResponse => {
        this.setCurrentUser({ currentUser: meResponse.data, csrf: response.data.csrf })
        this.error = ''
        this.$router.replace('/')
        this.$notify({
          title: `Sign ${this.signType} successful`,
          message: `You ${this.signType === 'up' ? 'signed up and' : ''} loged in successful`,
          type: 'success',
          duration: 2000
        })
      }).catch(error => this.signFailed(error))
    },
    signFailed (error) {
      this.error = (error.response && error.response.data && error.response.data.error) || 'Something went wrong'
      this.unsetCurrentUser()
    },
    checkSignedIn () {
      if (this.isSignedIn) {
        this.$router.replace('/')
      }
    },
    authorizationCode (authCode, provider) {
      this.$http.plain.post(`/oauth/${provider}`, { auth_code: authCode, provider: provider })
        .then(response => this.signSuccessful(response))
        .catch(error => this.signFailed(error))
    },
    signWithGoogle () {
      this.$gAuth.getAuthCode()
        .then(authCode => {
          this.authorizationCode(authCode, 'google')
        })
        .catch(error => this.signFailed(error))
    }
  }
}
</script>
