<template>
    <transition name="el-zoom-in-top">
        <div class="content-wrapper">

                                 <!-- Content Header (Page header) -->
        <section class="content-header">
            <h1>{{ getApplication.app_name }}<small><i class="ti-heart"></i><i class="ti-export"></i><i class="ti-printer"></i></small></h1>
            <ol class="breadcrumb">
                <li><router-link to="/"> <i class="ti-home"></i></router-link></li>
                <li><a href="#">Settings</a></li>
                <li class="active">User Management</li>
            </ol>
        </section>
            <!-- Main content -->
            <section class="content">
               <el-tabs v-model="activeName" @tab-click="loadInvitedUsers">
                  <el-tab-pane label="Users" name="usersList">
                    <div class="vld-parent">
                      <loading :active.sync="isLoading" 
                      :is-full-page="true"></loading></div>
                    <el-dialog
                      title="Invite User to your Organisation"
                      :visible.sync="isInvitation"
                      center>
                      <p type="warning" class="el-icon-warning" style="color: #E6A23C;" v-if="error"> {{ message }} </p><br/>
                      <invite-user v-on:send-invite="inviteUser" v-bind:user="userInfo" v-bind:loader="loading"
                          v-bind:searchEmail="searchEmail" v-on:search-user="searchUser" v-bind:roleName="roleName"
                          v-bind:hasDetails="hasDetails" v-bind:roles="roles" v-bind:callback="loadRoles"></invite-user>
                    </el-dialog>  
                    <div class="todo-devin">
                        <div class="row">
                            <div class="col-md-12">
                                <el-link icon="el-icon-plus" class="pull-right" style="padding: 0.8em;" @click="openInvite">Invite Existing User</el-link>
                                <el-table
                                  :data="users.filter(data => !search || data.first_name.toLowerCase().includes(search.toLowerCase()))"
                                  style="width: 100%">
                                  <el-table-column type="expand">
                                    <template slot-scope="scope">
                                      <div class="pull-left hidden-xs clearfix">
                                        <a href="#"><img v-bind:src="scope.row.image_url !== null ? scope.row.image_url : defaultImage"
                                          style="width: 80px; height: 83px; margin-right: 0.7em;" :alt="scope.row.id" /></a>
                                      </div>
                                      <div>
                                        <p>Name: {{ scope.row.first_name }} {{ scope.row.last_name }} {{ scope.row.other_name }}</p>
                                        <p>Email address: {{ scope.row.email_address }}</p>
                                        <p>Username: {{ scope.row.username }}</p>
                                      </div>                                
                                    </template>
                                  </el-table-column>
                                  <el-table-column
                                    label="First name"
                                    prop="first_name">
                                  </el-table-column>
                                  <el-table-column
                                    label="Last name"
                                    prop="last_name">
                                  </el-table-column>
                                  <el-table-column
                                    align="right">
                                    <template slot="header" slot-scope="scope">
                                      <el-input
                                        v-model="search"
                                        size="mini"
                                        placeholder="Type first name to search"/>
                                    </template>
                                    <template slot-scope="scope">
                                      <el-button
                                        size="mini" v-if="show" icon="el-icon-edit"
                                        @click="viewUserDetails(scope.row)">Edit</el-button>
                                      <el-button
                                        size="mini"
                                        type="danger" v-if="show" icon="el-icon-delete"
                                        @click="handleDelete(scope.$index, scope.row)">Delete</el-button>
                                    </template>
                                  </el-table-column>
                                </el-table>
                                <!-- <user-list v-bind:users="users"></user-list> -->
                            </div>
                        </div>
                    </div>
                  </el-tab-pane>
                  <el-tab-pane label="Invited Users" name="invitedUsers">
                    <div class="vld-parent">
                      <loading :active.sync="isLoading" 
                      :is-full-page="true"></loading></div>
                    <div class="todo-devin">
                        <div class="row">
                            <div class="col-md-12">
                                <el-table
                                  :data="invitedUsers.filter(data => !search || data.first_name.toLowerCase().includes(search.toLowerCase()))"
                                  style="width: 100%">                                 
                                  <el-table-column
                                    label="Name"
                                    prop="user_fullname">
                                  </el-table-column>
                                  <el-table-column
                                    label="Email Address"
                                    prop="user_email_address">
                                  </el-table-column>
                                  <el-table-column
                                    align="right">
                                    
                                    <template slot="header" slot-scope="scope">
                                      <el-input
                                        v-model="search"
                                        size="mini"
                                        placeholder="Type first name to search"/>
                                    </template>
                                    <template slot-scope="scope">
                                      <el-button  v-show="scope.row.status === 'pending'"
                                        size="mini" type="danger" round>Pending</el-button>
                                      <el-button  v-show="scope.row.status === 'approved'"
                                        size="mini" type="success" round>Accepted</el-button>
                                      <el-button v-if="scope.row.status === 'pending'" class="hidden-xs"
                                        size="mini" type="success" icon="el-icon-s-promotion"
                                        @click="resendInviteToUser(scope.row)">Resend Invite</el-button>                                      
                                      <el-button class="hidden-xs"
                                        size="mini"
                                        type="danger" v-if="scope.row.status === 'pending'" icon="el-icon-delete"
                                        @click="deleteInvitedUser(scope.row)">Revoke</el-button>
                                    </template>
                                  </el-table-column>
                                </el-table>
                                <!-- <user-list v-bind:users="users"></user-list> -->
                            </div>
                        </div>
                    </div>
                  </el-tab-pane>
                </el-tabs>
            </section>
            <!-- /.content -->
        </div>
    </transition>
</template>
<script>
import Loading from 'vue-loading-overlay'
import 'vue-loading-overlay/dist/vue-loading.css'
import { mapGetters } from 'vuex'
import sweetalert from 'sweetalert'
// import UserList from '../../settings/users/UserList.vue'
import CreateUser from '../../settings/users/CreateUser.vue'
import InviteUser from '../../settings/users/InviteUser.vue'
import { constants } from 'crypto';
export default {
  name: 'User',
  components: {
    CreateUser,
    Loading,
    InviteUser
  },
  data: function () {
    return {
      activeName: 'usersList',
      loading: false,
      message: '',
      error: false,
      show: false,
      status: false,
      isInvitation: false,
      search: '',
      userInfo: {},
      roles: [],
      hasDetails: false,
      searchEmail: '',
      isLoading: false,
      roleName: '',
      defaultImage: 'static/img/default_image.png',
      invitedUsers: [],
      users: [{
        id: 1,
        first_name: 'Murtadha',
        last_name: 'Ali',
        other_name: 'Olanrewaju',
        username: 'testuser',
        active: true
      }]
    }
  },
  mounted () {
    this.loadUsers(),
    this.checkRole()
  },
  computed: {
    ...mapGetters(['getUser', 'getToken', 'getApplication', 'getOrganisation', 'getRoles'])
  },
  methods: {
    openInvite: function () {
      this.message = ''
      this.error = false
      this.userInfo = {}
      this.hasDetails = false
      this.isInvitation = true
    },
    searchUser: function (param, callback) {
      console.log(this.loading)
      this.loading = true
      console.log(this.loading)
      this.$http.userapi.post('/users', null, {
        headers: {
          'search_parameter': param,
          'action': 'findUserByEmailOrUsername',
          'token': this.getToken
        }
      }).then(response => {
        this.loading = false
        console.log(response)
        if (response.data.code === 200) {
          this.userInfo = response.data.data
          if (this.userInfo) {
            if (callback) {
              callback()
            }
            this.hasDetails = true
            this.error = false
          }
        } else {
          this.error = true
          this.hasDetails = false
          this.message = response.data.message
        }
      }).catch(error => {
        this.loading = false
        console.log(error)
      })
    },
    loadRoles: function () {
      console.log(this.getRoles)
      if (this.getRoles.find(x => x.name === 'superadmin')) {
        this.$http.userapi.get('/roles', {
          headers: {
            'token': this.getToken
          }
        }).then(response => {
          if (response.data.code === 200) {
            const myRoles = response.data.data
            if (myRoles !== null && myRoles.length > 0) {
              this.roles = []
              for (var i = 0; i < myRoles.length; i++) {
                this.roles.push(myRoles[i])
              }
            }
          }
        }).catch(error => {
          console.log(error)
        })
      } else {
        this.loadRolesNotSuper()
      }
    },
    loadRolesNotSuper: function () {
      this.$http.userapi.post('/roles', null, {
        headers: {
          'token': this.getToken,
          'action': 'findRoleNotSuper'
        }
      }).then(response => {
        if (response.data.code === 200) {
          const myRoles = response.data.data
          if (myRoles !== null && myRoles.length > 0) {
            this.roles = []
            for (var i = 0; i < myRoles.length; i++) {
              this.roles.push(myRoles[i])
            }
          }
        }
      }).catch(error => {
        console.log(error)
      })
    },
    viewUserDetails: function (user) {
      console.log(user)
      this.$router.push('/user-details/' + user.username)
    },
    inviteUser: function (user, role) {
      this.loading = true
      this.$http.userapi.post('/users', null, {
        headers: {
          'email_address': user.email_address,
          'org_code': this.getOrganisation.code,
          'role_name': role,
          'action': 'inviteUser',
          'token': this.getToken
        }
      }).then(response => {
        this.loading = false
        console.log(response)
        if (response.data.code === 200) {
          this.isInvitation = false
          sweetalert('Invitation!', 'An invitation has been sent to\n' + user.email_address, 'success')
        }
      }).catch(error => {
        this.loading = false
        console.log(error)
      })
    },
    checkRole: function () {
      if (this.getRoles.find(x => (x.name === 'admin' || x.name === 'superadmin'))) {
        this.show = true
      } else {
        this.show = false
      }
      if (this.getRoles.find(x => (x.name === 'superadmin'))) {
        this.status = true
      } else {
        this.status = false
      }
    },
    createRole: function (newRole) {
      this.isLoading = true
      this.$http.userapi.post('/users', {
        'role_name': newRole.roleName,
        'description': newRole.description,
        'active': false,
        'created_by': this.getUser.username
      }, {
        headers: {
          'app_code': this.getApplication.app_code,
          'action': 'createRole',
          'token': this.getToken
        }
      }).then(response => {
        this.isLoading = false
        if (response.data.code === 200) {
          this.loadUsers()
          sweetalert('Success!', 'Role created!', 'success')
        }
      }).catch(error => {
        this.isLoading = false
        console.log(error)
      })
    },
    loadUsers: function () {
      this.isLoading = true
      this.$http.userapi.get('/users', {
        headers: {
          'action': 'findUsersByOrganisation',
          'org_code': this.getOrganisation.code,
          'token': this.getToken
        }
      }).then(response => {
        this.isLoading = false
        if (response.data.code === 200) {
          const myUsers = response.data.data
          if (myUsers !== null && myUsers.length > 0) {
            this.users = []
            for (var i = 0; i < myUsers.length; i++) {
              this.users.push(myUsers[i])
            }
          }
        }
      }).catch(error => {
        this.isLoading = false
        console.log(error)
      })
    },
    loadInvitedUsers: function (tab, event) {
      console.log('Calling invited users... ' + tab.name)
      if (tab.name === 'invitedUsers') {
        this.isLoading = true
        this.$http.userapi.get('/users', {
          headers: {
            'action': 'findInvitedUsers',
            'org_code': this.getOrganisation.code,
            'token': this.getToken
          }
        }).then(response => {
          this.isLoading = false
          if (response.data.code === 200) {
            console.log('Loaded ' + response.data.data)
            const myUsers = response.data.data
            if (myUsers !== null && myUsers.length > 0) {
              this.invitedUsers = []
              for (var i = 0; i < myUsers.length; i++) {
                this.invitedUsers.push(myUsers[i])
              }
            }
          }
        }).catch(error => {
          this.isLoading = false
          console.log(error)
        })
      }
    },
    resendInviteToUser: function (row) {
      this.isLoading = true
      this.$http.userapi.post('/users', null, {
        headers: {
          'action': 'resendInviteToUser',
          'request_code': row.code,
          'token': this.getToken
        }
      }).then(response => {
        this.isLoading = false
        console.log(response.data)
        if (response.data.code === 200) {
          sweetalert('Invitation!', response.data.message, 'success')
        }
      }).catch(error => {
        this.isLoading = false
        console.log(error)
      })
    },
    deleteInvitedUser: function (row) {
      this.$http.userapi.post('/users', null, {
        headers: {
          'action': 'deleteInvitedUser',
          'request_code': row.code,
          'token': this.getToken
        }
      }).then(response => {
        console.log(response.data)
        if (response.data.code === 200) {
          const rowIndex = this.invitedUsers.indexOf(row)
          if (rowIndex !== -1) {
            this.invitedUsers.splice(rowIndex, 1)
          }
        }
      }).catch(error => {
        console.log(error)
      })
    }
  }
}
</script>