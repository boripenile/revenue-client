<template>
  <div class="todo-list">
   <!-- <p >Completed Tasks: {{todos.filter(todo => {return todo.done === true}).length}}</p>
    <p >Pending Tasks: {{todos.filter(todo => {return todo.done === false}).length}}</p>-->
    <role v-on:delete-role="deleteRole" v-on:update-role="updateRole(role)" 
          v-on:activate-role="activateRole(role)" v-for="role in roles" :role.sync="role" :show="show"></role>
  </div>
</template>

<script>
import sweetalert from 'sweetalert'
import Role from './Role.vue'
import { mapGetters } from 'vuex'

export default {
  props: ['roles'],
  show: false,
  components: {
    Role
  },
  mounted () {
    this.checkRole()
  },
  computed: {
    ...mapGetters(['getToken', 'getUser', 'getRoles'])
  },
  methods: {
    deleteRole: function (role) {
    },
    checkRole () {
      if (this.getRoles.find(x => x.name === 'superadmin')) {
        this.show = true
      } else {
        this.show = false
      }
    },
    activateRole: function (role) {
      var enabled = true
      if (role.active) {
        enabled = false
      }
      this.$http.userapi.post('/roles', {
        'active': enabled,
        'updated_by': this.getUser.username
      }, {
        headers: {
          'id': role.id,
          'action': 'updateRole',
          'token': this.getToken
        }
      }).then(response => {
        if (response.data.code === 200) {
          var roleIndex = this.roles.indexOf(role)
          this.roles[roleIndex].active = enabled
          role = this.roles[roleIndex]
        }
      }).catch(error => {
        console.log(error)
      })
    },
    updateRole: function (role) {
      if (role.role_name.length > 0 && role.description.length > 0) {
        this.$http.userapi.post('/roles', {
          'role_name': role.role_name,
          'description': role.description,
          'active': role.active,
          'updated_by': this.getUser.username
        }, {
          headers: {
            'id': role.id,
            'action': 'updateRole',
            'token': this.getToken
          }
        }).then(response => {
          if (response.data.code === 200) {
            for (var i = 0; i < this.roles.length; i++) {
              if (this.roles[i].id === role.id) {
                this.roles[i].active = response.data.data.active
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



