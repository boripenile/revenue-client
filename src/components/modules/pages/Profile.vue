<template>
    <transition name="el-zoom-in-top">
        <div class="content-wrapper">
            <!-- Content Header (Page header) -->
            <section class="content-header">
                <h1>Profile Page<small>It's all start from here <i class="ti-heart"></i><i class="ti-export"></i><i class="ti-printer"></i></small></h1>
                <ol class="breadcrumb">
                    <li>
                        <router-link to="/"> <i class="ti-home"></i></router-link>
                    </li>
                    <li><a href="#">{{ getApplication.app_name }}</a></li>
                    <li class="active">Profile</li>
                </ol>
            </section>
            <!-- Main content -->
            <section class="content">
                <div class="box">
                    <div class="box-body" style="min-height:400px;">
                        <div class="col-lg-3">
                                <a href="#"><img v-bind:src="getUser.image_url !== null ? getUser.image_url : defaultImage"
                                width="190" height="200"
                                alt="cover" /></a>
                            <h2>{{ getUser.first_name }} {{ getUser.last_name }}
                                        <span class="pull-right social-profile">
                                            <i class="entypo-facebook-circled"></i>  <i class="entypo-twitter-circled"></i>  <i class="entypo-linkedin-circled"></i>  <i class="entypo-github-circled"></i>  <i class="entypo-gplus-circled"></i>
                                        </span>
                                    </h2>
                        </div>
                        <!-- end movie-card -->
                        <div class="col-lg-9 profile-name">
                            <dl class="dl-horizontal-profile"> <dt>Name</dt>
                                    <dd>{{ getUser.first_name }} {{ getUser.last_name }}</dd> <dt>Email</dt>
                                    <dd>{{ getUser.email_address }}</dd> <dt>Phone</dt>
                                    <dd>{{ getUser.phone_number }}</dd> <dt>Active Periode</dt>
                                    <dd>02 Dec 2014</dd> <dt>Last Update</dt>
                                    <dd>02 Apr 2014</dd>
                                </dl>
                          <!-- <permission-detail :permissions="permissions" :show="show"></permission-detail> -->
                        <div v-if="!superUser">
                          <h4 class="header">Basic Permisisons</h4>
                          <el-tag
                            :key="permission"
                            v-for="permission in basicPermissions" :closable="false" :disable-transitions="false">
                            {{permission.description}}
                          </el-tag>
                        </div>
                        <div v-if="!superUser">
                          <h4 class="header">User Permisisons</h4>
                          <el-tag
                            :key="permission"
                            v-for="permission in selfPermissions" :closable="false" :disable-transitions="false">
                            {{permission.description}}
                          </el-tag>
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
  import { mapGetters, mapMutations } from 'vuex'
  export default {
    data: function () {
      return {
        loading: false,
        defaultImage: 'static/img/default_image.png',
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
    name: 'Profile',
    computed: {
      ...mapGetters(['getUser', 'getApplication', 'getOrganisation', 'getRoles', 'getPermissions', 'getToken'])
    },
    components: {
    },
    mounted () {
      this.checkRole(),
      this.isSuper(),
      this.isAdmin(),
      this.basicPermissions = this.getPermissions.basic,
      this.selfPermissions = this.getPermissions.self
    },
    methods: {
      ...mapMutations(['setUserId']),
      showPermissions: function () {
        this.centerDialogVisible = true
        this.loading = true
        this.loadPermissions()
      },
      removePermission: function (permission) {
        return;
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
