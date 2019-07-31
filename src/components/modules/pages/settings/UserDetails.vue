<template>
    <transition name="el-zoom-in-top">
        <div class="content-wrapper">
            <div class="vld-parent">
              <loading :active.sync="isLoading" 
              :is-full-page="true"></loading></div>
            <!-- Content Header (Page header) -->
            <section class="content-header">
                <h1>Profile Page<small>It's all start from here <i class="ti-heart"></i><i class="ti-export"></i><i class="ti-printer"></i></small></h1>
                <ol class="breadcrumb">
                    <li>
                        <router-link to="/"> <i class="ti-home"></i></router-link>
                    </li>
                    <li><a href="#">{{ getOrganisation.name }}</a></li>
                    <li><a href="#"><router-link to="/users">Users</router-link></a></li>
                    <li class="active">Profile</li>
                </ol>
            </section>
            <!-- Main content -->
            <section class="content">
                <div class="box">
                    <div class="box-body" style="min-height:400px;">
                        <div class="col-lg-3">
                                <a href="#"><img v-bind:src="user.image_url !== null ? user.image_url : defaultImage"
                                width="190" height="200"
                                alt="cover" /></a>
                            <h2>{{ user.first_name }} {{ user.last_name }}
                                        <span class="pull-right social-profile">
                                            <i class="entypo-facebook-circled"></i>  <i class="entypo-twitter-circled"></i>  <i class="entypo-linkedin-circled"></i>  <i class="entypo-github-circled"></i>  <i class="entypo-gplus-circled"></i>
                                        </span>
                                    </h2>
                        </div>
                        <!-- end movie-card -->
                        <div class="col-lg-9 profile-name">
                            <dl class="dl-horizontal-profile"> <dt>Name</dt>
                                    <dd>{{ user.first_name }} {{ user.last_name }}</dd> <dt>Email</dt>
                                    <dd>{{ user.email_address }}</dd> <dt>Phone</dt>
                                    <dd>{{ user.phone_number }}</dd> <dt>Active Periode</dt>
                                    <dd>02 Dec 2014</dd> <dt>Last Update</dt>
                                    <dd>{{ user.updated_at }}</dd>
                                </dl>
                          <!-- <permission-detail :permissions="permissions" :show="show"></permission-detail> -->
                        <div v-if="user.id > 1">
                          <h4 class="header">Basic Permisisons</h4>
                          <hr/>
                          <el-tag
                            :key="permission"
                            v-for="permission in basicPermissions" :closable="false" :disable-transitions="false" @close="removePermission(permission)">
                            {{permission.description}}
                          </el-tag>
                        </div>
                        <div v-if="user.id > 1">
                          <h4 class="header">User Permisisons</h4>
                          <el-link icon="el-icon-edit" @click="showPermissions">Update Self Permissions</el-link>
                          <hr/>
                          <el-tag
                            :key="permission"
                            v-for="permission in selfPermissions" :closable="show" :disable-transitions="false" @close="removePermission(permission)">
                            {{permission.description}}
                          </el-tag><br/>
                          <el-dialog
                            title="Add Permissions"
                            :visible.sync="centerDialogVisible"
                            center>
                            <div class="scroll">
                              <el-table ref="multipleSelection"
                                v-loading="loading"
                                :data="permissions"
                                @selection-change="handleSelectionChange">
                                <el-table-column
                                  type="selection"
                                  width="45">
                                </el-table-column>
                                <el-table-column
                                  property="description"
                                  label="Description"
                                  show-overflow-tooltip>
                                </el-table-column>
                              </el-table>
                            </div>
                            <span slot="footer" class="dialog-footer">
                              <el-button @click="centerDialogVisible = false">Cancel</el-button>
                              <el-button type="success" @click="addPermissionsToUser">Confirm</el-button>
                            </span>
                          </el-dialog>
                        </div>
                        </div>
                    </div>
                </div>
            </section>
            <!-- /.content -->
        </div>
    </transition>
</template>
<script>
  import Loading from 'vue-loading-overlay'
  import 'vue-loading-overlay/dist/vue-loading.css'
  import { mapGetters, mapMutations } from 'vuex'
  export default {
    data: function () {
      return {
        isLoading: false,
        loading: false,
        user: {},
        roles: [],
        permissions: {},
        username: this.$route.params.username,
        defaultImage: '.././static/img/default_image.png',
        basicPermissions: [],
        selfPermissions: [],
        show: false,
        superUser: false,
        adminUser: false,
        permissions: [],
        multipleSelection: [],
        userPermissions: [],
        selection: {},
        centerDialogVisible: false
      }
    },
    name: 'UserDetails',
    components: {
      Loading
    },
    mounted () {
      this.loadUserDetails(this.setRoleAndPermissions),
      this.checkRole(),
      this.isSuper(),
      this.isAdmin()
    },
    computed: {
      ...mapGetters(['getUser', 'getApplication', 'getOrganisation', 'getRoles', 'getPermissions', 'getToken'])
    },
    methods: {
      setRoleAndPermissions: function () {
        console.log(this.permissions.basic)
        console.log(this.permissions.self)
        this.basicPermissions = this.permissions.basic
        this.selfPermissions = this.permissions.self
      },
      loadUserDetails: function (callback) {
        if (this.username != null) {
          this.isLoading = true
          this.$http.userapi.post('/users', {
            'username': this.username
          }, {
            headers: {
              'token': this.getToken,
              'org_code': this.getOrganisation.code,
              'action': 'findUserDetails'
            }
          }).then(response => {
            this.isLoading = false
            if (response.data.code === 400) {
              this.message = response.data.message
              this.$router.push('/users')
            } else {
              this.roles = []
              this.permissions = {}
              this.user = response.data.data
              // this.roles = response.data.roles
              this.permissions = response.data.permissions
              if (callback) {
                console.log('Started...')
                console.log(this.permissions)
                callback()
              }
            }
          }).catch(error => {
            this.isLoading = false
            console.log(error)
          })
        }
      },
      showPermissions: function () {
        this.centerDialogVisible = true
        this.loading = true
        this.loadPermissions()
      },
      removePermission: function (permission) {
        this.$http.userapi.post('/roles', {
          'orgCode': this.getOrganisation.code,
          'userIds': [this.user.id],
          'permissionNames': [permission.name]
        }, {
          headers: {
            'token': this.getToken,
            'action': 'removePermissionsFromUsers' 
          }
        }).then(response => {
          console.log(response.data)
          const permissionIndex = this.selfPermissions.indexOf(permission)
          this.selfPermissions.splice(permissionIndex, 1)
          this.loading = false
        }).catch(error => {
          this.loading = false
          console.log(error)
        })
      },
      isSuper: function () {
        if (this.getRoles.find(x => (x.name === 'superadmin'))) {
          this.superUser = true
        } else {
          this.superUser = false
        }
      },
      isAdmin: function () {
        if (this.getRoles.find(x => (x.name === 'admin'))) {
          this.adminUser = true
        } else {
          this.adminUser = false
        }
      },
      checkRole: function () {
        if (this.getRoles.find(x => (x.name === 'superadmin' || x.name === 'admin'))) {
          this.show = true
        } else {
          this.show = false
        }
      },
      loadPermissions: function () {
        this.$http.userapi.post('/permissions', null, {
          headers: {
            'token': this.getToken,
            'action': 'findPermissionNotSuper' 
          }
        }).then(response => {
          this.permissions = []
          for (var i = 0; i < response.data.data.length - 1; i++) {
            this.permissions.push(response.data.data[i])
          }
          this.loading = false
        }).catch(error => {
          this.loading = false
          console.log(error)
        })
      },
      tableRowClassName({row, rowIndex}) {
        if (rowIndex % 2 === 0) {
          return 'success-row';
        }
        return '';
      },
      handleSelectionChange: function (val) {
        this.multipleSelection = val
        console.log(this.multipleSelection)
      },
      addPermissionsToUser: function () {
        this.userPermissions = []
        for (var i = 0; i < this.multipleSelection.length; i++ ) {
          console.log(this.multipleSelection[i].permission_name)
          this.userPermissions.push(this.multipleSelection[i].permission_name)
        }
        this.processPermissionForUser(this.includePermissions)
        this.centerDialogVisible = false
      },
      includePermissions: function () {
        if (this.multipleSelection.length > 0) {
          for (var j = 0; j < this.multipleSelection.length; j++) {
            var record = {
              'name': this.multipleSelection[j].permission_name,
              'description': this.multipleSelection[j].description
            }
            const found = this.selfPermissions.find(x => x.name === record.name)
            if (!found) {
              this.selfPermissions.push(record)
            }
          }
        }
      },
      processPermissionForUser: function (callback) {
        this.loading = true
        this.$http.userapi.post('/roles', {
          'orgCode': this.getOrganisation.code,
          'userIds': [this.user.id],
          'permissionNames': this.userPermissions
        }, {
          headers: {
            'token': this.getToken,
            'action': 'addPermissionsToUsers' 
          }
        }).then(response => {
          console.log(response.data)
          if (callback) {
            callback()
          }
          this.loading = false
        }).catch(error => {
          this.loading = false
          console.log(error)
        })
      }
    }
  }
</script>
<style>
  .el-tag + .el-tag {
    margin-left: 10px;
    margin-bottom: 10px;
  }
  .scroll {
    height: 400px;
    overflow: auto;
  }
  .float-right {
    float: right;
  }
</style>
