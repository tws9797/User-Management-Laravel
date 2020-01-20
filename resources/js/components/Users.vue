<template>
    <div class="container">
        <div class="row mt-5" v-if="$gate.isAdminOrAuthor()">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal">Add New <i class="fas fa-user-plus fa-fw"></i></button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody>
                    <tr>
                        <th>ID</th>
                        <th>Name</th>
                        <th>Email</th>
                        <th>Type</th>
                        <th>Registered At</th>
                        <th>Modify</th>
                  </tr>

                  <tr v-for="user in users.data" :key="user.id">

                    <td>{{user.id}}</td>
                    <td>{{user.name}}</td>
                    <td>{{user.email}}</td>
                    <td>{{user.type | upText}}</td>
                    <td>{{user.created_at | myDate}}</td>

                    <td>
                        <a href="#" @click="editModal(user)">
                            <i class="fa fa-edit blue"></i>
                        </a>
                        /
                        <a href="#" @click="deleteUser(user.id)">
                            <i class="fa fa-trash red"></i>
                        </a>

                    </td>
                  </tr>
                </tbody></table>
              </div>
              <div class="card-footer">
                <pagination :data="users" @pagination-change-page="getResults"></pagination>
              </div>
            </div>
            <!-- /.card -->
          </div>
        </div>

        <div v-if="!$gate.isAdminOrAuthor()">
            <not-found></not-found>
        </div>

    <!-- Modal -->
            <div class="modal fade" id="addNewModal" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" v-show="!editMode" id="addNewLabel">Add New</h5>
                    <h5 class="modal-title" v-show="editMode" id="addNewLabel">Update User's Info</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <form @submit.prevent="editMode ? updateUser() : createUser()">
                <div class="modal-body">
                     <div class="form-group">
                        <input v-model="form.name" type="text" name="name"
                            placeholder="Name"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                        <has-error :form="form" field="name"></has-error>
                    </div>

                     <div class="form-group">
                        <input v-model="form.email" type="email" name="email"
                            placeholder="Email Address"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                        <has-error :form="form" field="email"></has-error>
                    </div>

                     <div class="form-group">
                        <textarea v-model="form.bio" name="bio" id="bio"
                        placeholder="Short bio for user (Optional)"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                        <has-error :form="form" field="bio"></has-error>
                    </div>


                    <div class="form-group">
                        <select name="type" v-model="form.type" id="type" class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                            <option value="">Select User Role</option>
                            <option value="admin">Admin</option>
                            <option value="user">Standard User</option>
                            <option value="author">Author</option>
                        </select>
                        <has-error :form="form" field="type"></has-error>
                    </div>

                    <div class="form-group">
                        <input v-model="form.password" type="password" name="password" id="password"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                        <has-error :form="form" field="password"></has-error>
                    </div>

                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                    <button v-show="editMode" type="submit" class="btn btn-success">Update</button>
                    <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>
                </div>

                </form>

                </div>
            </div>
            </div>
    </div>

</template>

<script>
import Form from 'vform'

    export default {
        data() {
          return {
            editMode: true,
            users: {},
            form: new Form({
              id: '',
              name: '',
              email: '',
              password: '',
              type: '',
              bio: '',
              photo: ''
            })
          }
        },
        methods: {
          getResults(page = 1) {
      			axios.get('api/user?page=' + page)
      				.then(response => {
      					this.users = response.data;
      				});
    		  },
          updateUser(id){
            this.$Progress.start();
            this.form.put('api/user/' + this.form.id)
            .then(() => {
              $('#addNewModal').modal('hide');
              swal.fire("Updated!", "Information has been updated", "success")
              this.$Progress.finish();
              fire.$emit('updateList');
            })
            .catch(() => {
              this.$Progress.fail();
            });
          },
          editModal(user){
            this.editMode = true;
            this.form.reset();
            $('#addNewModal').modal('show');
            this.form.fill(user);
          },
          newModal(){
            this.editMode = false;
            this.form.reset();
            $('#addNewModal').modal('show');
          },
          deleteUser(id){
            swal.fire({
              title: 'Are you sure?',
              text: "You won't be able to revert this!",
              icon: 'warning',
              showCancelButton: true,
              confirmButtonColor: '#3085d6',
              cancelButtonColor: '#d33',
              confirmButtonText: 'Yes, delete it!'
            }).then((result) => {
              if(result.value){
                this.form.delete('api/user/' + id).then(() => {
                  swal.fire(
                    'Deleted!',
                    'Your file has been deleted.',
                    'success'
                  )
                  fire.$emit('updateList');
                }).catch(() => {
                  swal.fire("Failed", "There was something wrong.", "warning")
                });
              }
            })
          },
          loadUsers(){
              if(this.$gate.isAdminOrAuthor()){
                axios.get("api/user").then(({ data }) => (this.users = data));
              }
          },
          createUser(){
            this.$Progress.start();
            this.form.post('api/users')
            .then(() => {
              fire.$emit('updateList');
              $('#addNewModal').modal('hide');
              toast.fire({
                icon: 'success',
                title: 'User created successfully'
              });
              this.$Progress.finish();
            })
            .catch(() => {
              this.$Progress.fail();
            });
          }
        },
        created() {
            fire.$on('searching', () => {
              let query = this.$parent.search;
              axios.get('api/findUser?q=' + query)
              .then((data) => {
                this.users = data.data;
              })
              .catch();
            });
            this.loadUsers();
            fire.$on('updateList', () => {
              this.loadUsers();
            });
        }
    }
</script>
