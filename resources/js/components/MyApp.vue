<template>
    <div class="user">
        <img src="https://img.freepik.com/free-psd/3d-illustration-human-avatar-profile_23-2150671142.jpg?size=338&ext=jpg&ga=GA1.1.1395880969.1709596800&semt=ais"
            alt="">
        <div class="user-txt">
            <h6>Hello Rex,</h6>
            <p>Create and organize your tasks</p>
        </div>
    </div>
    <div class="container cnt">
        <form @submit.prevent="submitForm" class="form-cnt">
            <div class="mb-3 mt-4">
                <label for="title" class="form-label">Title</label>
                <input type="text" v-model="formData.title" class="form-control" id="title" required>
            </div>
            <div class="mb-3">
                <label for="exampleInputPassword1" class="form-label">Task Description</label>
                <textarea class="form-control" v-model="formData.description" id="" cols="30" rows="5"
                    required></textarea>
            </div>


            <button v-if="isAdding" type="button" class="btn btn-secondary">Adding Task <span
                    class="spinner-border-sm spinner-border" role="status">
                </span></button>

            <button v-else type="submit" class="btn btn-secondary">Add Task </button>
        </form>

        <div v-if="isLoading" class="loading">
            <span class="spinner-border" role="status"></span>
        </div>

        <div v-else class="tsk-list">

            <div v-if="tasks.length > 0">
                <div v-for="t in tasks" :key="t.id">
                    <div class="tsk-item">
                        <div @click="handleCheck(t.id)" class="form-check">
                            <input class="form-check-input" type="checkbox" value="" :id="t.id" :checked="t.completed">
                            <label class="form-check-label" :for="t.id">
                                {{ t.title }}
                            </label>
                        </div>

                        <!-- <p>{{ t.completed == 0 ? 'not done' : 'completed' }}</p> -->
                        <div @click="delTask(t.id)" class="btn btn-danger text-white btn-sm">
                            Delete <i class="fa-solid fa-trash"></i>
                        </div>
                    </div>
                </div>

            </div>
            <div v-else>
                <div class="alert alert-info">No Task has been created yet.</div>
            </div>

        </div>


    </div>
</template>

<script>

import axios from 'axios';
import toastr from 'toastr';
// import 'toastr/toastr.css';


export default {
    name: "MyApp",

    data() {
        return {
            tasks: [],
            isLoading: true,
            isAdding: false,
            formData: {
                title: '',
                description: ''
            }
        }
    },
    created() {
        // Initialize Toastr
        toastr.options = {
            positionClass: 'toast-top-right',
            preventDuplicates: true,
            closeButton: true,
            progressBar: true,
            timeOut: 6000
        };
    },
    mounted() {
        axios.get('http://localhost:8000/api/tasks')
            .then(response => {
                this.isLoading = false
                this.tasks = response.data;
                // console.log(response.data)
            })
            .catch(error => {
                this.isLoading = false
                console.log(error)
            });
    },
    methods: {
        submitForm() {
            this.isAdding = true
            axios.post('http://localhost:8000/api/tasks', this.formData)
                .then(response => {
                    this.isAdding = false;
                    toastr.success("Task added")
                    console.log('Response:', response.data);
                    this.tasks = [response.data, ...this.tasks];
                    this.resetForm();
                })
                .catch(error => {
                    toastr.error("Error occured while creating this task.")
                    this.isAdding = false
                    console.error('Error:', error);
                    this.error = error.response.data.message;
                });
        },
        delTask(id) {
            this.tasks = [...this.tasks.filter(task => task.id !== id)]
            axios.delete(`http://localhost:8000/api/tasks/${id}`, this.formData).then((data) => {
                console.log('success');
            })
                .catch(error => {
                    toastr.error("Error occured while Deleting this task.")
                    this.isAdding = false
                    console.error('Error:', error);
                    this.error = error.response.data.message;
                });
        },
        resetForm() {
            this.formData.title = '';
            this.formData.description = '';
        },
        handleCheck(id) {
            console.log(id)
            axios.put(`http://localhost:8000/api/tasks/${id}`, this.formData).then((data) => {
                console.log('success');
            })
                .catch(error => {
                    toastr.error("Error occured while Deleting this task.")
                    this.isAdding = false
                    console.error('Error:', error);
                    this.error = error.response.data.message;
                });
        }
    }
}
</script>