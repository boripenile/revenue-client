<template>
  <div class="todo-list">
   <!-- <p >Completed Tasks: {{todos.filter(todo => {return todo.done === true}).length}}</p>
    <p >Pending Tasks: {{todos.filter(todo => {return todo.done === false}).length}}</p>-->
    <user v-on:remove-user-from-organisation="removeUserFromOrganisation" v-for="user in users" :user.sync="user"
      v-on:update-user="updateUser(user)" 
      v-on:activate-user="activateUser(user)" 
      :show="show" :status="status" v-on:view-details="viewUserDetails(user)"></user>
  </div>
</template>

<script>
import sweetalert from 'sweetalert'
import User from './User.vue'
import { mapGetters } from 'vuex'

export default {
  props: ['users'],
  data: function () {
    return {
      show: false,
      status: false
    }
  },
  components: {
    User
  },
  mounted () {
    this.checkRole()
  },
  computed: {
    ...mapGetters(['getToken', 'getUser', 'getRoles', 'getOrganisation'])
  },
  methods: {
    checkRole () {
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
    removeUserFromOrganisation: function (user) {
      sweetalert({
        title: 'Are you sure?',
        text: 'This User will be removed permanently',
        type: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#DD6B55',
        confirmButtonText: 'Yes, remove it!',
        closeOnConfirm: false
      },
      () => {
        this.$http.userapi.post('/users', null, {
          headers: {
            'id': user.id,
            'action': 'deleteRole',
            'token': this.getToken,
            'org_code': this.getOrganisation.code
          }
        }).then(response => {
          if (response.data.code === 200) {
            const userIndex = this.users.indexOf(user)
            this.users.splice(userIndex, 1)
            sweetalert('Deleted!', 'User has been deleted.', 'success')
          }
        }).catch(error => {
          console.log(error)
        })
      })
    },
    activateRole: function (user) {
      var enabled = true
      if (user.active) {
        enabled = false
      }
      this.$http.userapi.post('/users', {
        'active': enabled,
        'updated_by': this.getUser.username
      }, {
        headers: {
          'id': user.id,
          'action': 'updateUser',
          'token': this.getToken
        }
      }).then(response => {
        if (response.data.code === 200) {
          var userIndex = this.users.indexOf(user)
          this.users[userIndex].active = enabled
          user = this.users[userIndex]
        }
      }).catch(error => {
        console.log(error)
      })
    },
    viewUserDetails: function (user) {
      console.log(user)
      this.$router.push('/user-details/' + user.username)
    },
    updateRole: function (user) {
      if (user.role_name.length > 0 && user.description.length > 0) {
        this.$http.userapi.post('/users', {
          'first_name': user.first_name,
          'last_name': user.last_name,
          'other_name': user.last_name,
          'active': user.active,
          'updated_by': this.getUser.username
        }, {
          headers: {
            'id': user.id,
            'action': 'updateUser',
            'token': this.getToken
          }
        }).then(response => {
          if (response.data.code === 200) {
            for (var i = 0; i < this.users.length; i++) {
              if (this.users[i].id === user.id) {
                this.users[i].active = response.data.data.active
              }
            }
          }
        }).catch(error => {
          console.log(error)
        })
      }
    }
  }
}
</script>



