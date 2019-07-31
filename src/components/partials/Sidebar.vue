<template>
    <aside class="main-sidebar">
        <!-- sidebar: style can be found in sidebar.less -->
        <div>
            <section class="sidebar">
                <!-- /.search form -->
                <!-- Sidebar user panel -->
                <div class="user-panel">
                    <div class="image"> <img src="static/img/avatar2.png" class="img-circle" alt="User Image"> </div>
                    <div class="info">
                        <p>Welcome, {{ getUser.username }}</p>
                        <p><small>You haven't miss any task this week!</small></p>
                        <div> <a href="#"><i class="ti-panel"></i><small>Settings</small></a> <a href="#" @click="profile()"><i class="ti-user"></i><small>Profile</small></a> <a href="#" @click="logout()"><i class="ti-power-off"></i><small>Logout</small></a> </div>
                    </div> <img class="bg-user" src="static/img/avatar-bg.png" alt="User Image"> </div>
                <!-- sidebar menu: : style can be found in sidebar.less -->
                <ul class="sidebar-menu">
                    <li class="header">APPS</li>
                    <li class="treeview" v-if="show">
                        <a href="#"> <i class="ti-desktop"></i> <span>Dashboard</span> <span class="pull-right-container">
              <i class="fa fa-angle-left pull-right"></i>
            </span> </a>
                        <ul class="treeview-menu">
                            <li>
                                <router-link to="/dashboard">
                                    <avatar username="Dashaboard One" :size='20' color="#fff"></avatar>Dashboard 1</router-link>
                            </li>
                        </ul>
                    </li>
                    <li class="treeview"  v-if="show">
                        <a href="#"> <i class="ti-panel"></i> <span>Settings</span> <span class="pull-right-container">
              <i class="fa fa-angle-left pull-right"></i>
            </span> </a>
                        <ul class="treeview-menu">
                            <li>
                                <router-link to="/roles">
                                    <avatar username="Roles Management" :size='20' color="#fff"></avatar>Roles Management</router-link>
                            </li>
                            <li>
                                <router-link to="/users">
                                    <avatar username="Users Management" :size='20' color="#fff"></avatar>Users Management</router-link>
                            </li>
                        </ul>
                    </li>
                    <li>
                        <br>
                        <br>
                        <br>
                    </li>
                </ul>
            </section>
            <!-- /.sidebar -->
        </div>
    </aside>
</template>
<script>
import Avatar from 'vue-avatar'
import { mapMutations, mapGetters } from 'vuex'
export default {
  name: 'DashboardSidebar',
  data: function () {
    return {
      show: false
    }  
  },
  mounted: function () {
    $(document).ready(function ($) {
      $('.main-sidebar > div').slimScroll({
        width: '230px',
        position: 'left',
        size: '5px',
        height: '95vh'
      })
    }),
    this.checkRole()
  },
  components: {
    Avatar
  },
  computed: {
    ...mapGetters(['getUser', 'getRoles', 'getOrganisation'])
  },
  methods: {
    ...mapMutations(['setToken', 'setUser', 'setOrganisation']),
    logout: function () {
      this.setToken(null)
      this.setUser(null)
      this.setOrganisation(null)
      this.$router.push('/app/login')
    },
    checkRole: function () {
      if (!this.getOrganisation) {
        this.show = false
      } else {
        if (this.getRoles.find(x => (x.name === 'superadmin' || x.name === 'admin'))) {
          this.show = true
        } else {
          this.show = false
        }
      }
    },
    profile: function () {
      this.$router.push('/profile')
    }
  }
}
</script>