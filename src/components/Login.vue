<template>
  <transition name="el-zoom-in-top">
      <div class="login-box">
        <center><h2>Login</h2></center>
        <hr>
        <form role="form">
          <div class="form-group">
            <a href="#" class="pull-right label-forgot">Forgot email?</a>
            <label for="inputUsernameEmail">Username or email</label>
            <input type="text" id="inputUsernameEmail" v-model="username" class="form-control">
          </div>
          <div class="form-group">
            <a href="#" class="pull-right label-forgot">Forgot password?</a>
            <label for="inputPassword">Password</label>
            <input type="password" id="inputPassword" v-model="password" class="form-control">
          </div>
          <el-button :loading="loading" :disabled="username.length < 5 || password.length < 6" type="primary" round class="btn btn btn-primary pull-right" 
          @click.prevent="login">Log In</el-button>
        </form>
        <a class="forgotLnk" href="#"></a>
        <div class="or-box">
          <center>
            <span class="text-center login-with">
              <b>Sign Up to Become</b>
            </span>
          </center>
        </div>
        <div class="row-block">
          <div class="row">
            <div class="col-md-6">
              <el-link class="pull-left" type="warning" @click="registrationPage()"><i class="el-icon-s-shop" style="font-size: 20px;"></i> Provider</el-link>  
            </div>
            <div class="col-md-6">
              <el-link class="pull-right" type="danger" @click="userRegistrationPage()"><i class="el-icon-user-solid" style="font-size: 20px;"></i> User</el-link>
            </div>
          </div>
        </div>
        <div class="row-block">
          <div class="row">
            
          </div>
        </div>
      </div>
  </transition>
</template>
<script>
import { mapMutations } from 'vuex'
import sweetalert from 'sweetalert'

export default {
  data: function () {
    return {
      username: '',
      password: '',
      loading: false
    }
  },
  methods: {
    ...mapMutations(['setUser', 'setToken', 'setApplication', 'setRoles', 'setPermissions', 'setOrganisations', 'setOrganisation']),
    showAlert: function (message) {
      sweetalert({
        title: 'Error',
        text: message,
        type: 'error',
        confirmButtonColor: '#DD6B55',
        confirmButtonText: 'OK',
        closeOnConfirm: true
      })
    },
    registrationPage: function () {
      this.$router.push('/app/registration')
    },
    userRegistrationPage: function () {
      this.$router.push('/app/user-reg')
    },
    login: function () {
      const username = this.username
      const password = this.password
      if (username.length > 0 && password.length > 0) {
        this.loading = true
        this.$http.userapi.post('/login', {
          'username': username,
          'password': password
        }, {
          headers: {
            'app_code': this.$store.state.appCode
          }
        }).then(response => {
          this.loading = false
          if (response.data.code === 400) {
            this.showAlert(response.data.message)
          } else {
            if (!response.data.organisation) {
              if (!response.data.token) {
                this.showAlert('You have not activated your account')
              } else {
                console.log(response.data)
                this.setUser(response.data.data)
                this.setToken(response.data.token)
                this.setApplication(response.data.application)
                this.$notify.success({
                  title: 'Login',
                  message: 'Logged in successfully',
                  showClose: false
                })
                this.$router.push('/profile')
              }
            } else {
              if (response.data.organisation.length === 1) {
                this.setToken(response.data.token)
                this.setUser(response.data.data)
                this.setApplication(response.data.application)
                this.setOrganisation(response.data.organisation[0])
                this.setRoles(response.data.roles)
                this.setPermissions(response.data.permissions)
                this.$notify.success({
                  title: 'Login',
                  message: 'Logged in successfully',
                  showClose: false
                })
                this.$router.push('/profile')
              } else {
                this.setUser(response.data.data)
                this.setApplication(response.data.application)
                this.setOrganisations(response.data.organisation)
                this.$router.push('/app/my-companies')
              }
            }
          }
        }).catch(error => {
          this.loading = false
          console.log(error)
        })
      }
    }
  }
}
</script>

<style>
.login-box {
  margin: auto;
}
</style>
