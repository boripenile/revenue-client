<template>
 <div class="vld-parent">
          <loading :active.sync="isLoading" 
          :is-full-page="true"></loading>
    <transition name="el-zoom-in-top">
      <div class="box">
        <div class="box-body">
          <div class="row content-center">
            <div class="col-sm-8 col-lg-6 col-xs-12">
              <el-link @click="loginPage()" type="info" icon="el-icon-d-arrow-left">Back to Login</el-link>
                <!-- <h3><b>User Registration</b></h3> -->
                <br/>
                <el-alert v-if="success"
                  v-bind:title="title"
                  v-bind:type="type"
                  v-bind:description="message"
                  show-icon>
                </el-alert>
              <el-steps :active="active" align-center>
                <el-step title="Step 1" description="Basic Information" icon="el-icon-user"></el-step>
                <el-step title="Step 2" description="Login Information" icon="el-icon-setting"></el-step>
              </el-steps>
            </div>
          </div>
        </div>
      </div>
    </transition>
      <el-form v-if="active === 0" :rules="basicRules" :model="basicInfo" ref="basicInfoForm" status-icon>
        <div class="box">
          <div class="box-body">
            <h4 class="content-center"><b>Basic Information</b></h4>
            <p class="content-center">Please complete all fields expect optional ones.<br/>
            Ensure that you use a valid email address and phone number</p>
            <div class="row content-center">
              <div class="col-sm-8 col-lg-6 col-xs-12">
                <p>First Name</p>
                <el-form-item prop="firstName">
                  <el-input v-model="basicInfo.firstName" placeholder="First name"></el-input>
                </el-form-item>
              </div></div>
              <div class="row content-center">
              <div class="col-sm-8 col-lg-6 col-xs-12">
                <p>Last Name</p>
                <el-form-item prop="lastName">
                  <el-input v-model="basicInfo.lastName" placeholder="Last name"></el-input>
                </el-form-item>
              </div></div>
              <div class="row content-center">
              <div class="col-sm-8 col-lg-6 col-xs-12">
                <p>Email Address</p>
                <el-form-item prop="emailAddress">
                  <el-input v-model="basicInfo.emailAddress" placeholder="Enter a valid Email Address"></el-input>
                </el-form-item>
              </div></div>
              <div class="row content-center">
              <div class="col-sm-8 col-lg-6 col-xs-12">
                <p>Phone Number</p>
                <el-form-item prop="phoneNumber">
                  <el-input v-model.number="basicInfo.phoneNumber" placeholder="Enter a valid phone number" type="number"></el-input>
                </el-form-item>
              </div></div>
            <br/>
              <div class="row content-center">
                <!-- <div class="col-sm-8 col-lg-6 col-xs-12"> -->
                  <el-button round class="btn btn"
          @click.prevent="resetBasicRegistrationForm('basicInfoForm')">Reset Fields</i></el-button>
                <el-button type="primary" round class="btn btn btn-primary" 
          @click.prevent="validateBasicRegistration('basicInfoForm')">Complete Login details <i class="el-icon-right"></i></el-button>
                <!-- </div> -->
              </div>        
          </div>
        </div>
    </el-form>
    <el-form v-if="active === 1" :rules="rules" :model="loginInfo" ref="loginInfoForm" status-icon>
      <div class="box">
            <div class="box-body">
              <div class="row content-center">
                <div class="col-sm-8 col-lg-6 col-xs-12">
                  <el-link icon="el-icon-d-arrow-left" @click="active = 0">Back</el-link>
                </div>
              </div>
          <h4 class="content-center"><b>Login Information</b></h4>
              <p class="content-center">Please complete all fields expect optional ones</p>
              <div class="row content-center">
                <div class="col-sm-8 col-lg-6 col-xs-12">
                  <p>Username</p>
                  <el-form-item prop="username">
                    <el-input v-model="loginInfo.username" placeholder="Choose a username"></el-input>
                  </el-form-item>
                </div></div>
                <div class="row content-center">
                <div class="col-sm-8 col-lg-6 col-xs-12">
                  <p>Password</p>
                  <el-form-item prop="password">
                    <el-input v-model="loginInfo.password" placeholder="Enter password" type="password"></el-input>
                  </el-form-item>
                </div></div>
                <div class="row content-center">
                <div class="col-sm-8 col-lg-6 col-xs-12">
                  <p>Confirm Password</p>
                  <el-form-item prop="confirmPassword">
                    <el-input v-model="loginInfo.confirmPassword" placeholder="Confirm password" type="password"></el-input>
                  </el-form-item>
                </div></div>
              <br/>
              <div class="row content-center">
              <div class="pull-right">
                <el-button round class="btn btn" 
          @click.prevent="resetLoginRegistrationForm('loginInfoForm')">Reset</i></el-button>
                <el-button type="primary" round class="btn btn btn-primary left-margin" 
          @click.prevent="validateLoginRegistration('loginInfoForm')">Proceed with Registration <i class="el-icon-d-arrow-right"></i></el-button>
              </div>
              </div>
          </div>
        </div>
    </el-form>
 </div>
</template>
<script>
import Loading from 'vue-loading-overlay'
import 'vue-loading-overlay/dist/vue-loading.css'
import { mapMutations, mapGetters } from 'vuex'
import sweetalert from 'sweetalert'

export default {
  data: function () {
    var validateEmailAddress = (rule, value, callback) => {
      if (value === '') {
        return callback(new Error('Provide a valid email address'))
      } else if (value !== '') {
        if (this.regex.test(value) === false) {
          callback(new Error('Invalid email address. Please check it'))
        } else {
          callback()
        }
      }
    }
    var validatePhoneNumber = (rule, value, callback) => {
      if (!value) {
        return callback(new Error('Provide a valid phone number'))
      } else if (!Number.isInteger(value)) {
        callback(new Error('Invalid phone number. Please check it'))
      } else {
        var numberStr = String(value)
        if (numberStr.length === 13) {
          callback()
        } else {
          callback(new Error('Phone number must be 13 characters. Example: 2348078675634'))
        }
      }
    }
    var validatePass = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('Please choose a password'))
      } else {
        if (value.length < 8) {
          callback(new Error('Password must be at least 8 characters'))
        }
        if (this.loginInfo.confirmPassword !== '') {
          this.$refs.loginInfoForm.validateField('confirmPassword')
        }
        callback()
      }
    }
    var validateConfirmPass = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('Please confirm your password'))
      } else if (value !== this.loginInfo.password) {
        callback(new Error('Confirm password does not match with the password! Please re-enter it.'))
      } else {
        callback()
      }
    }
    return {
      regex: /^(([^<>()\]\\.,;:\s@"]+(\.[^<>()\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,24}))$/,
      isLoading: false,
      success: false,
      title: '',
      type: '',
      active: 0,
      message: 'Error message',
      basicInfo: {
        firstName: '',
        lastName: '',
        emailAddress: '',
        phoneNumber: ''
      },
      loginInfo: {
        username: '',
        password: '',
        confirmPassword: ''
      },
      basicRules: {
        firstName: [
          { required: true, message: 'Provide your administrator\'s first name', trigger: 'blur' },
          { min: 3, max: 20, message: 'First name should be 3 to 20 characters.', trigger: 'blur' }
        ],
        lastName: [
          { required: true, message: 'Provide your administrator\'s last name', trigger: 'blur' },
          { min: 3, max: 20, message: 'Last name should be 3 to 20 characters.', trigger: 'blur' }
        ],
        emailAddress: [
          { validator: validateEmailAddress, trigger: 'blur' }
        ],
        phoneNumber: [
          { validator: validatePhoneNumber, trigger: 'blur' }
        ]
      },
      rules: {
        username: [
          { required: true, message: 'Choose a username', trigger: 'blur' },
          { min: 7, max: 50, message: 'Username should be 7 to 50 characters.', trigger: 'blur' }
        ],
        password: [
          { validator: validatePass, trigger: 'blur' }
        ],
        confirmPassword: [
          { validator: validateConfirmPass, trigger: 'blur' }
        ]
      }
    }
  },
  components: {
    Loading
  },
  mounted () {
  },
  methods: {
    validateBasicRegistration: function (basicInfoForm) {
      this.$refs[basicInfoForm].validate((valid) => {
        if (valid) {
          this.active = 1
        } else {
          return false
        }
      })
    },
    validateLoginRegistration: function (loginInfoForm) {
      this.$refs[loginInfoForm].validate((valid) => {
        if (valid) {
          this.registerUser()
        } else {
          return false
        }
      })
    },
    resetBasicRegistrationForm: function (basicInfoForm) {
      this.$refs[basicInfoForm].resetFields()
    },
    resetLoginRegistrationForm: function (loginInfoForm) {
      this.$refs[loginInfoForm].resetFields()
    },
    loginPage: function () {
      this.$router.push('/app/login')
    },
    registerUser: function () {
      this.isLoading = true
      console.log('Saving user...')
      this.$http.userapi.post('/users', {
        'firstName': this.basicInfo.firstName,
        'lastName': this.basicInfo.lastName,
        'emailAddress': this.basicInfo.emailAddress,
        'phoneNumber': this.basicInfo.phoneNumber,
        'username': this.loginInfo.username,
        'password': this.loginInfo.password,
        'appCode': this.$store.state.appCode
      }, {
        headers: {
          'action': 'registerUser'
        }
      }).then(response => {
        this.isLoading = false
        if (response.data.code === 400) {
          this.success = true
          this.type = 'warning'
          this.title = 'Registration cannot be completed'
          this.message = response.data.message
        } else {
          if (response.data.code === 200) {
            this.success = true
            this.type = 'success'
            this.title = 'Registration completed successfully'
            this.message = response.data.message
            this.resetBasicRegistrationForm('basicInfoForm')
            this.resetLoginRegistrationForm('loginInfoForm')
          }
        }
      }).catch(error => {
        this.isLoading = false
        console.log(error)
      })
    }
  }
}
</script>

<style>
.left-margin {
  margin-right: 4em;
}
.content-center {
  display: flex;
  align-items: center;
  justify-content: center;
}
.content-left {
  display: flex;
  align-items: center;
  justify-content: left;
}
</style>
