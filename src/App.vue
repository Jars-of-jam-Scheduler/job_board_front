
<template>
    <div>
      <h1>Jf Boadrd</h1>
testf
        <!-- Login popup -->
        <div id="loginPopup" v-if="showLoginPopup">
        <h3>Login</h3>
        <form @submit.prevent>
          <input type="text" placeholder="Username" v-model="username" />
          <input type="password" placeholder="Password" v-model="password" />
          <button @click="login">Login</button>
          <button @click="toggleLoginPopup">Cancel</button>
        </form>
      </div>

      <button @click="toggleLoginPopup">Login</button>
  
      <ul>
        <li v-for="(job, index) in jobs" :key="index">
          <h3>{{ job.title }}</h3>
          <p>{{ job.description }}</p>
        </li>
      </ul>
      
    </div>
</template>
  
<script>
import axios from 'axios';

export default {
  data() {
    return {
      jobs: [],
      page: 1,
      isLoading: false,
      showLoginPopup: false,
      username: '',
      password: '',
    }
  },
  mounted() {
    this.fetchJobs();
    window.addEventListener('scroll', this.handleScroll);
  },
  beforeUnmount() {
    window.removeEventListener('scroll', this.handleScroll);
  },
  methods: {
    fetchJobs() {
      // Only fetch jobs if we're not already loading
      if (this.isLoading) {
        return;
      }
      this.isLoading = true;

      axios
        .get('http://localhost:80/api/jobs?page=' + this.page)
        .then(response => {
          this.jobs = [...this.jobs, ...response.data.data];
          this.page++;
          this.isLoading = false;
        })
        .catch(error => {
          console.log(error);
          this.isLoading = false;
        });
    },
    handleScroll() {
      const bottom =
        document.documentElement.scrollHeight -
        (window.innerHeight + window.scrollY);
      if (bottom <= 250) {
        this.fetchJobs();
      }
    },
    toggleLoginPopup() {
      this.showLoginPopup = !this.showLoginPopup;
    },
    login() {
        axios
        .post('http://localhost:80/api/sanctum/token', {
            email: this.username,
            password: this.password,
            device_name: 'webapp_device'
        })
        .then(response => {
            // Login was successful, save the token to use for future requests
            localStorage.setItem('token', response.data.token);
            this.showLoginPopup = false;
        })
        .catch(error => {
            // Login failed, display an error message to the user
            alert('Login failed, please try again');
            console.error(error)
        });
  }
  }
}
</script>