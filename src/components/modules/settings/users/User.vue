<template>
    <div class="todo-content">
        <div v-show="!isEditing">
            <div class="pull-left right-padding hidden-xs clearfix">
              <a href="#"><img v-bind:src="user.image_url !== null ? user.image_url : defaultImage"
                width="80" height="83" alt="user.id" /></a>
            </div>
            <h2>{{ user.first_name }} {{ user.last_name }} {{ user.other_name }}</h2>
            <div class="task-action"> <span v-on:click="viewDetails">
          <i class='ti-pencil'></i>
        </span> <span v-if="show" v-on:click="removeUserFromOrganisation(user)">
          <i class='ti-trash'></i>
        </span> </div>
        </div>
        <div class="content-todo-devin" v-show="isEditing">
            <div>
                <div>
                    <label>First Name</label>
                    <p>
                        <el-input placeholder="First name" v-model="user.first_name" type='text'></el-input>
                    </p>
                </div>
                <div>
                    <label>Last Name</label>
                    <p>
                        <el-input placeholder="Last name" v-model="user.last_name" type='text'></el-input>
                    </p>
                </div>
                <div>
                    <label>Other Name</label>
                    <p>
                        <el-input placeholder="Other name" v-model="user.other_name" type='text'></el-input>
                    </p>
                  <input type="hidden" v-model="user.active">
                </div>
                <div>
                    <el-button v-on:click="hideForm" type="danger">Close</el-button>
                    <el-button v-on:click="updateUser(user)" type="success">Update</el-button>
                </div>
            </div>
        </div>
        <div class="todo-completed" v-if="status" v-on:click="activateUser(user)" v-show="!isEditing && user.active"> Enabled </div>
        <div class="todo-pending" v-if="status" v-on:click="activateUser(user)" v-show="!isEditing && !user.active"> Disabled </div>
    </div>
</template>
<script>
  export default {
    props: ['user', 'show', 'status'],
    data: function () {
      return {
        isEditing: false,
        defaultImage: './static/img/default_image.png'
      }
    },
    methods: {
      activateUser: function (user) {
        this.$emit('activate-user', user)
      },
      removeUserFromOrganisation: function (user) {
        this.$emit('remove-user-from-organisation', user)
      },
      updateUser: function (user) {
        this.$emit('update-user', user)
        this.hideForm()
      },
      viewDetails: function (user) {
        this.$emit('view-details', user)
        this.hideForm()
      },
      showForm: function () {
        this.isEditing = true
      },
      hideForm: function () {
        this.isEditing = false
      }
    }
  }
</script>
<style>
  .right-padding {
    padding-right: 1.3em;
  }
</style>
