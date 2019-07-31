<template>
    <div>
      <div class="row" style="text-align: center;">
        <div class="col-md-9">
          <el-input
            placeholder="Type user's email address or username"
            v-model="searchEmail">
          </el-input>
        </div>
        <div class="col-md-3">
          <el-button :disabled="searchEmail.length < 6" :loading="loader" type="default" icon="el-icon-search" @click="searchUser(searchEmail, callback)">Look Up</el-button>
        </div>
      </div>
      <hr v-if="hasDetails" />
      <div class="row" v-if="hasDetails">
        <div class="col-md-5" style="float: left">
          <p><b>Name</b></p><p>{{ user.first_name }} {{ user.last_name }} {{ user.other_name }} </p>
          <p><b>Email address</b></p><p>{{ user.email_address }}</p>
        </div>
        <div class="col-md-7">
          <p><b>Select a role to assign to the user</b></p>
          <el-select v-model="roleName" placeholder="Select">
            <el-option
              v-for="role in roles"
              :key="role.role_name"
              :label="role.description"
              :value="role.role_name">
              <span style="float: left; color: #8492a6; font-size: 13px">{{ role.description }}</span>
            </el-option>
          </el-select>
          <el-button :disabled="roleName === ''" :loading="loader" type="success" style="margin-left: 20px;" icon="el-icon-s-promotion" @click="sendInvite(user, roleName)">Send Invite</el-button>
        </div>
      </div> 
      
    </div>
    
</template>

<script>
export default {
  props: ['user', 'roles', 'searchEmail', 'hasDetails', 'callback', 'roleName', 'loader'],
  name: 'InviteUser',
  data: function () {
    return {
      hasDetails: false
    }
  },
  methods: {
    sendInvite: function (user, roleName) {
      this.$emit('send-invite', user, roleName)
    },
    searchUser: function (param, callback) {
      if (param) {
        this.$emit('search-user', param, callback)
      }
    }
  } 
}
</script>
