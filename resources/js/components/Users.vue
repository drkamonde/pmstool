<template>
    <div class="container">
        <div class="row mt-5" v-if="$gate.isAdminOrOperation()">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal">Add New 
                      <i class="fas fa-user-plus fa-fw"></i>
                    </button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody><tr>
                    <th>ID</th>
                    <th>User</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Register at</th>
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
              <!-- /.card-body -->
                <div class="card-footer">
                  <pagination :data="users" @pagination-change-page="getResults">
                      <span slot="prev-nav">&lt; Previous</span>
	                    <span slot="next-nav">Next &gt;</span>
                  </pagination>
                </div>
            </div>
            <!-- /.card -->
          </div>
        </div>

        <div v-if="!$gate.isAdminOrOperation()">
          <not-found></not-found>
        </div>
        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
          <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" v-show="!editMode" id="addNew">Add New</h5>
                <h5 class="modal-title" v-show="editMode" id="addNew">Update user's Info</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>
              <form @submit.prevent="editMode ? updateUser(): createUser()">
                <div class="modal-body">

                <div class="form-group">                
                    <input v-model="form.name" type="text" name="name"
                      placeholder="Name"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                    <has-error :form="form" field="name"></has-error>
                </div>

                <div class="form-group">                
                    <input v-model="form.email" type="email" email="email"
                      placeholder="Email Address"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                    <has-error :form="form" field="email"></has-error>
                </div>

                <div class="form-group">                
                    <textarea v-model="form.bio" type="text" name="bio" id="bio"
                      placeholder="Short bio for user (Optional)"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                    <has-error :form="form" field="name"></has-error>
                </div>

                <div class="form-group">
                    <select name="type" v-model="form.type" id="type" class="form-control" :class="{'is-invalid': form.errors.has('type') }">
                      <option value="">Select User Role</option>
                      <option value="admin">Admin</option>
                      <option value="operation">Operation</option>
                      <option value="finance">Finance</option>
                      <option value="hr">HR</option>
                      <option value="pmanager">Project Manager</option>
                      <option value="tleader">Team Leader</option>
                      <option value="agent">Agent</option>
                      <option value="driver">Driver</option>
                    </select>
                    <has-error :form="form" field="type"></has-error>
                </div>

                <div class="form-group">                
                    <input v-model="form.password" type="password" name="name" id="password"                      
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
    export default {
        data() {
          return{
            editMode: false,
            users : {},
            form: new Form({
              id: '',
              name: '',  
              email: '',
              password: '',
              type: '',
              bio: '',
                   
            })
          }
        },
        methods: {
          getResults(page = 1){
            axios.get('api/user?page=' + page)
				    .then(response => {
				    	this.users = response.data;
				    });
          },

          updateUser(){
            this.$Progress.start();
            //console.log('Editing data');
            this.form.put('api/user/'+this.form.id)
            .then(() =>{
              //success
              $('#addNew').modal('hide')
              Swal.fire(
                'Updated!',
                'Information has been updated.',
                'success'
              ) 
              Fire.$emit('AfterCreate')
              this.$Progress.finish();              

            })
            .catch(() =>{
              this.$Progress.fail(); 
            })
          },
          editModal(user){
            this.editMode = true;
            this.form.reset();
            $('#addNew').modal('show');
            this.form.fill(user);
          },
          newModal(){
            this.editMode = false;
            this.form.reset();
            $('#addNew').modal('show');
          },

          deleteUser(id){
              Swal.fire({
                title: 'Are you sure?',
                text: "You won't be able to revert this!",
                type: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Yes, delete it!'
                }).then((result) => {

                //send request to the serve
                if (result.value) {
                    this.form.delete('api/user/'+id).then(()=>{                  
                        Swal.fire(
                          'Deleted!',
                          'Your file has been deleted.',
                          'success'
                        ) 
                      Fire.$emit('AfterCreate')                
                    }).catch(()=> {
                      Swal.fire("Failed","There was something wrong.", "warning");
                    });
                }
                
              })
          },
          loadUsers(){
            if(this.$gate.isAdminOrOperation()){    
                axios.get('api/user').then(({ data }) => (this.users = data));
            }
          },

          createUser(){
            this.$Progress.start();

            this.form.post('api/user')
            .then(()=>{
                Fire.$emit('AfterCreate');

                $('#addNew').modal('hide')

                toast.fire({
                  type: 'success',
                  title: 'User Created successfully'
                })

                this.$Progress.finish(); 

            })
            .catch(()=>{

            })


          }
        },
        created() {
            this.loadUsers();
            Fire.$on('AfterCreate',() => {
              this.loadUsers();
            })
            //setInterval(() => this.loadUsers(), 3000);
        }
    }
</script>