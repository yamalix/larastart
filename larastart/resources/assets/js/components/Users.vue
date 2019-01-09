<template>
    <div class="container">
        <div class="row mt-5" v-if="$gate.isAdmin()">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users Table</h3>

                        <div class="card-tools">
                            <button class="btn btn-success" @click="openModal">Add New <i class="fa fa-user-plus fa-fw"></i></button>
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
                                <td>{{user.id }}</td>
                                <td>{{user.name | upText}}</td>
                                <td>{{user.email}}</td>
                                <td>{{user.type | upText}}</td>
                                <td>{{user.created_at | myDate}}</td>
                                <td>
                                    <a href="#" class="mr-2" @click="editModal(user)">
                                    <i class="fa fa-edit">

                                    </i>
                                    </a>
                                    <a href="#" @click="deleteUser(user.id)">
                                        <i  class="fa fa-trash ">

                                        </i>
                                    </a>
                                </td>
                            </tr>

                            </tbody></table>
                    </div>
                    <!-- /.card-body -->
                    <div class="card-footer">
                        <pagination :data="users" @pagination-change-page="getResults"></pagination>

                    </div>
                </div>
                <!-- /.card -->
            </div>
        </div>

        <!--modal-->

        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewTitle" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 v-show="!editMode" class="modal-title" id="addNewTitle">Add New User</h5>
                        <h5 v-show="editMode" class="modal-title" id="updateTitle">Update User</h5>

                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode ? updateUser(form.id) : createUser()">
                    <div class="modal-body">
                        <div class="form-group">
                            <input placeholder="name" v-model="form.name" type="text" id="name" name="name"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                            <has-error :form="form" field="name"></has-error>
                        </div>
                        <div class="form-group">
                            <input placeholder="Email Address" v-model="form.email" id="email" type="text" name="email"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                            <has-error :form="form" field="email"></has-error>
                        </div>
                        <div class="form-group">
                            <textarea placeholder="short bio for user (Optional)" id="bio" v-model="form.bio" type="text" name="bio"
                                      class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                            <has-error :form="form" field="bio"></has-error>
                        </div>
                        <div class="form-group">
                            <select  id="type" v-model="form.type" type="type" name="type"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">

                            <option value="">Select User Role</option>
                                <option value="admin">Admin</option>
                                <option value="user"> Standard User</option>
                                <option value="auhtor">Author</option>

                            </select>
                            <has-error :form="form" field="type"></has-error>
                        </div>
                        <div class="form-group">
                                <input placeholder="Password" v-model="form.password" id="password" type="text" name="password"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                                <has-error :form="form" field="password"></has-error>
                            </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                        <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>
                        <button v-show="editMode" type="submit" class="btn btn-info">Update</button>

                    </div>
                    </form>
                </div>

                </div>
            </div>
        </div>

</template>

<script>
    export default {

        methods:{
            getResults(page = 1) {
                axios.get('api/user?page=' + page)
                    .then(response => {
                        this.users = response.data;
                    });},
            updateUser(){
                this.$Progress.start();

                this.form.patch('api/user/'+ this.form.id)
                        .then(()=> {
                            Fire.$emit('AfterCreated');
                            $('#addNew').modal('hide');
                            toast({
                                type: 'success',
                                title: 'User updated '
                            });

                            this.$Progress.finish();

                        })
                        .catch(()=> {
                            this.$Progress.start();

                        })
            },
            editModal(user){
                this.editMode=true;
                this.form.reset();
                $('#addNew').modal('show');
                this.form.fill(user)
            },
            openModal(){
                this.editMode=false;
                this.form.reset();
                $('#addNew').modal('show');

            },
            createUser(){
                console.log("creating");
                this.editMode=false;
                this.$Progress.start();
                this.form.post('api/user')
                    .then(()=>{
                        Fire.$emit('AfterCreated');
                        this.$Progress.finish();
                        $('#addNew').modal('hide');
                        toast({
                            type: 'success',
                            title: 'User created '
                        });
                    })
                    .catch(()=>{
                    });
             //   setTimeout(this.loadUsers, 3000);
            },
            deleteUser(id){

                swal({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                        this.form.delete('api/user/' + id)
                            .then(()=>{
                                    swal(
                                        'Deleted!',
                                        'Your file has been deleted.',
                                        'success'
                                    )
                                Fire.$emit('AfterCreated');

                            }).catch(()=>{
                                swal("Failed", "something is wrong", "warning")
                        } );
                    //send ajax request to the server

                })
            },
            loadUsers(){
                axios.get("api/user").then(({data})=>(this.users=data));
            }
        },
        data(){
            return{
                editMode: false,
                users: {},
                form: new Form({
                    id:'',
                    name:'',
                    password:'',
                    email:'',
                    type:'',
                    bio:'',
                    photo:''
                })
            }
        },
        created() {
            this.loadUsers();
            Fire.$on('AfterCreated',()=>{
                this.loadUsers();
            })
        }
    }
</script>
