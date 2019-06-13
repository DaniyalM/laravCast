<template>
    <div class="container">
        <div class="row mt-5">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users List</h3>

                        <div class="card-tools">

                            <button class="btn btn-success" @click="openModal">
                                <i class="fa fa-user-plus fa-fw"></i>
                                Add New
                            </button>
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
                                <th>Created AT</th>
                                <th>Modify</th>
                            </tr>
                            <tr v-for="user in users" :key="user.id">

                                <td>{{user.id}}</td>
                                <td>{{user.name}}</td>
                                <td>{{user.email}}</td>
                                <td>{{user.type | upText}}</td>
                                <td>{{user.created_at|myDate}}</td>
                                <!--<td><span class="tag tag-success">{{user.bio}}</span></td>-->
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

                            </tbody>
                        </table>
                    </div>
                    <!-- /.card-body -->
                </div>
                <!-- /.card -->
            </div>
        </div>
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel"
             aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 v-show="!editmode" class="modal-title" >Add New</h5>
                        <h5 v-show="editmode" class="modal-title" >Update</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editmode? updateUser() :createUser()" @keydown="form.onKeydown($event)">
                        <div class="modal-body">

                            <div class="form-group">

                                <input v-model="form.name" type="text" name="name" placeholder="Name"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
                            </div>
                            <div class="form-group">

                                <input v-model="form.email" type="email" name="email" placeholder="Email"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                                <has-error :form="form" field="email"></has-error>
                            </div>
                            <div class="form-group">

                            <textarea v-model="form.bio" type="text" name="bio" placeholder="Short Bio for User"
                                      class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }">

                            </textarea>
                                <has-error :form="form" field="bio"></has-error>
                            </div>

                            <div class="form-group">

                                <select v-model="form.type" type="text" name="type"
                                        class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                    <option value="">Select User Role</option>
                                    <option value="admin">Admin</option>
                                    <option value="user">Standard User</option>
                                    <option value="author">Author</option>

                                </select>
                                <has-error :form="form" field="type"></has-error>
                            </div>
                            <div class="form-group">

                            <textarea v-model="form.password" type="password" name="password" placeholder="Password"
                                      class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">

                            </textarea>
                                <has-error :form="form" field="password"></has-error>
                            </div>

                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal" @click="loadUsers">Close
                            </button>
                            <button v-show="editmode" type="submit" class="btn btn-success">Update</button>
                            <button v-show="!editmode" type="submit" class="btn btn-primary">Create</button>
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
            return {
                editmode: false,
                users: {},
                form: new Form({
                    id:'',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: ''
                })
            }
        },
        mounted() {
            // alert('Component mounted.');
        },
        methods: {
            editModal(user) {
                this.form.reset();
                this.editmode=true;
                $('#addNew').modal('show')
                this.form.fill(user)
            },
            openModal() {
                this.form.reset();
                this.editmode=false;
                $('#addNew').modal('show')
            }
            ,
            updateUser() {
                this.$Progress.start()
                this.form.put('api/user/' + this.form.id).then(()=>{
                    $('#addNew').modal('hide')
                    this.$Progress.finish()
                    swal.fire(
                        'Updated!',
                        'User has been Updated Successfully !.',
                        'success'
                    )
                    Fire.$emit('afterCreated');

                }).catch(()=>{
                    this.$Progress.fail()
                })
            }
            ,
            createUser() {
                this.$Progress.start()

                this.form.post('api/user').then(() => {
                    $('#addNew').modal('hide')
                    Fire.$emit('afterCreated');
                    toast.fire({
                        type: 'success',
                        title: 'User Created successfully'
                    })
                    this.$Progress.finish()
                });


            },
            loadUsers() {
                axios.get("api/user").then(({data}) => (this.users = data.data));

            }
            ,
            deleteUser(id) {
                swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    if (result.value) {
                        this.form.delete('api/user/' + id).then(() => {

                            swal.fire(
                                'Deleted!',
                                'Your file has been deleted.',
                                'success'
                            )
                            Fire.$emit('afterCreated');


                        }).catch(() => {
                            swal(
                                "Failed", "May be Network issue.", "warning"
                            )
                        })
                    }


                })
            }
        },
        created() {
            this.loadUsers();
            // setInterval(()=>this.loadUsers(),3000);
            Fire.$on('afterCreated', () => this.loadUsers());
        }
    }
</script>
