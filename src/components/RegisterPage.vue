<template>
  <div class="container mt-5 w-25">
    <h4>Registration Form</h4>
    <form @submit.prevent="registerUser">
      <div class="form-group mb-3">
        <input type="text" class="form-control" id="name" v-model="name" @input="clearErrors('name')" placeholder="Enter name" required>
      </div>
      <div class="form-group mb-3">
        <input type="email" class="form-control" id="email" v-model="email" @input="clearErrors('email')" placeholder="Enter email" required>
        <small class="text-danger" v-if="errors.email">{{ errors.email[0] }}</small>
      </div>
      <div class="form-group mb-3">
        <input type="password" class="form-control" id="password" v-model="password" @input="clearErrors('password')" placeholder="Password" required>
        <small class="text-danger" v-if="errors.password">{{ errors.password[0] }}</small>
      </div>
      <div class="form-group mb-3">
        <input type="password" class="form-control" id="confirm" v-model="confirm" placeholder="Confirm Password" required>
      </div>
      <button type="submit" class="btn btn-primary mt-3 w-100">Register</button>
    </form>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  data() {
    return {
      name: '',
      email: '',
      password: '',
      confirm: '',
      errors: {}
    };
  },
  methods: {
    async registerUser() {
        try {
        // Call the register API endpoint using Axios
        // the /register is an API route that define in the backend using Laravel
        const response = await axios.post(this.$root.$data.apiUrl + '/register', {
          name: this.name,
          email: this.email,
          password: this.password,
          password_confirmation: this.confirm
        });
        if (response.status === 201) {
            // After successful registration, handle redirection or other actions
            // Clear the form fields after successful registration
            this.name = '';
            this.email = '';
            this.password = '';
            this.confirm = '';
            alert('Registration successful');
            // Redirect to the login page
            this.$router.push('/');
        }
      } catch (error) {
          // Handle registration error, show error message to the user
        this.errors = error.response.data.errors;
      }
    },
    //the field param is the name of the input field that triggered the error
    clearErrors(field) {
      // Clear error message for the specified field
      // this method is called when the user types in the input field triggering the event handler 
      this.errors[field] = null;
    }
  }
};
</script>

<style>
    body {
        background-image: url(https://img.freepik.com/free-photo/abstract-gradient-neon-lights_23-2149279180.jpg);
        background-size: cover;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        margin: 0;
    }

    .container {
        background-color: #fff;
        padding: 20px;
        border-radius: 10px;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    }

    h4 {
        color: rgb(58, 148, 103);
        text-align: center;
    }

    .form-group {
        position: relative;
    }

    .form-control {
        border-radius: 5px;
        padding: 1rem;
    }

    .btn-primary {
        background-color:rgb(58, 150, 104);
        border: none;
        border-radius: 5px;
        padding: 0.75rem;
        font-size: 1rem;
    }

    .btn-primary:hover {
        background-color: #035322;
    }

    .text-danger {
        position: absolute;
        bottom: -1.5rem;
        left: 0;
        font-size: 0.875rem;
    }
</style>