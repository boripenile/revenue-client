<template>
  <transition name="el-zoom-in-top">
      <div class="login-box">
        <div class="vld-parent">
          <loading :active.sync="isLoading" 
          :is-full-page="true"></loading></div>
        <center>
          <i class="el-icon-success" v-if="success" style="font-size: 6em; color: #67C23A;"></i>
          <i class="el-icon-error" v-if="error" style="font-size: 6em; color: #F56C6C;"></i>

          <h2>Request Approval</h2>
          <br/>
          <h4>{{ message }}</h4>
          <br/>
          <el-button type="success" 
              @click="loginPage()">Login</el-button>
        </center>
        <br/><br/><br/>
        <br/><br/><br/><br/>  
      </div>
  </transition>
</template>
<script>
import Loading from 'vue-loading-overlay'
import 'vue-loading-overlay/dist/vue-loading.css'
import { mapMutations, mapGetters } from 'vuex'
import sweetalert from 'sweetalert'

export default {
  data: function () {
    return {
      isLoading: false,
      success: false,
      error: false,
      message: 'Approving pending request, please wait...',
      verifyCode: this.$route.params.code
    }
  },
  components: {
    Loading
  },
  mounted () {
    this.approvePendingRequest()
  },
  computed: {
    ...mapGetters(['getOrganisation'])
  },
  methods: {
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
    loginPage: function () {
      this.$router.push('/app/login')
    },
    approvePendingRequest: function () {
      if (this.verifyCode != null) {
        this.isLoading = true
        this.$http.userapi.post('/users', null, {
          headers: {
            'verify_code': this.verifyCode,
            'action': 'approveUserRequest'
          }
        }).then(response => {
          this.isLoading = false
          if (response.data.code === 400) {
            this.error = true
            this.message = response.data.message
          } else {
            this.success = true
            this.message = 'You have successful approved the request.'
          }
        }).catch(error => {
          this.isLoading = false
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
