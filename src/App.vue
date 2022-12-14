
<template>
	<div>
		<h1>Job Board</h1>

		<div v-if="user">
			<p>Logged in as {{ user.name }}</p>
			<p>Your e-mail is: {{ user.email }}</p>
			<p>Your roles are: {{ user.translated_roles.join(', ') }}</p>
			<button @click="logout">Log out</button>
		</div>

		<!-- Login popup -->
		<div id="loginPopup" v-if="showLoginPopup">
			<h2>Login</h2>
			<form @submit.prevent>
				<input type="text" placeholder="Username" v-model="username" />
				<input type="password" placeholder="Password" v-model="password" />
				<button @click="login">Login</button>
				<button @click="toggleLoginPopup">Cancel</button>
			</form>
		</div>

		<button v-if="!user && !showLoginPopup" @click="toggleLoginPopup">Login</button>

		<SoftDeletedJobs :softDeletedJobs="softDeletedJobs" @restoredJobOffer="fetchJobs(true); fetchSoftDeletedJobs();" v-if="user && user.roles.includes('firm')" />
	
		<h2>Jobs affichés</h2>
			<!-- Add Job Offer Form -->
			<AddJobForm @addedJobOffer="addJobOffer" v-if="user && user.roles.includes('firm')" />
		<ul>
			<li v-for="(job, index) in jobs" :key="index">
				<h3>{{ job.title }}</h3>
				<p>{{ job.description }}</p>
				<button v-if="user && user.roles.includes('firm')" @click="deleteJob(index)">Delete</button>
			</li>
		</ul>

	</div>
</template>
  
<script>
import axios from 'axios';
import AddJobForm from '@/components/AddJobForm';
import SoftDeletedJobs from '@/components/SoftDeletedJobs';

export default {
	components: {
		AddJobForm,
		SoftDeletedJobs
	},
	data() {
		return {
			jobs: [],
			softDeletedJobs: [],
			page: 1,
			isLoading: false,
			showLoginPopup: false,
			username: '',
			password: '',
			user: null,
			localStorage: localStorage
		}
	},
	mounted() {
		if (localStorage.getItem('token')) {
			this.fetchUserData();			
		} else {
			this.fetchJobs(true);
		}
	},
	beforeUnmount() {
		if (typeof this.handleScroll === 'function') {
			window.removeEventListener('scroll', this.handleScroll);
		}
	},
	methods: {
		fetchJobs(forcePageOne = false) {
			if (this.isLoading) {  // Only fetch jobs if we're not already loading
				return;
			}
			this.isLoading = true;

			if (forcePageOne) {
				this.page = 1;
				this.jobs = [];
				window.addEventListener('scroll', this.handleScroll);
			}

			let url = 'http://localhost:80/api/jobs?page=' + this.page;
			if (this.user && this.user.roles.includes('firm')) {
				url = 'http://localhost:80/api/firm/jobs?page=' + this.page;
			}

			axios.get(url, {
				'headers': {
					Authorization: `Bearer ${localStorage.getItem('token')}`,
				}
			})
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
			axios.post('http://localhost:80/api/sanctum/token', {
				email: this.username,
				password: this.password,
				device_name: 'webapp_device'
			})
				.then(response => {
					localStorage.setItem('token', response.data);
					this.showLoginPopup = false;
					this.fetchUserData();
				})
				.catch(error => {
					alert('Login failed, please try again');
					console.error(error)
				});
		},
		logout() {
			axios.post('http://localhost:80/api/user/logout', {}, {
				'headers': {
					Authorization: `Bearer ${localStorage.getItem('token')}`,
				}
			})
				.then(() => {
					localStorage.removeItem("token");
					this.user = null;

					this.fetchJobs(true);
					window.addEventListener('scroll', this.handleScroll);
				})
				.catch(error => {
					alert('Logout failed, please try again');
					console.error(error)
				});
		},
		fetchUserData() {
			axios.get('http://localhost:80/api/user', {
				'headers': {
					Authorization: `Bearer ${localStorage.getItem('token')}`,
				}
			})
				.then(response => {
					this.user = response.data.user;

					this.fetchJobs(true);
					window.addEventListener('scroll', this.handleScroll);

					if(this.user.roles.includes('firm')) {
						this.fetchSoftDeletedJobs();
					}
				})
				.catch(error => {
					alert('Fetch user profile failed, please try again');
					console.error(error);
				});
		},
		addJobOffer(job_offer) {
			this.jobs = [...this.jobs, job_offer];
		},
		deleteJob(index) {
			axios.delete('http://localhost:80/api/jobs/' + this.jobs[index].id, {
				headers: {
					'Authorization': `Bearer ${localStorage.getItem('token')}`,
				}
			}).then(() => {
				this.jobs.splice(index, 1);
				this.fetchSoftDeletedJobs();
			}).catch(error => {
				alert('Job deletion failed, please try again');
				console.error(error);
			});
		},
		fetchSoftDeletedJobs() {
			axios.get('http://localhost:80/api/firm/soft_deleted_jobs', {
					'headers': {
						Authorization: `Bearer ${localStorage.getItem('token')}`,
					}
				})
			.then(response => {
				this.fetchJobs();
				this.softDeletedJobs = response.data.data
			})
			.catch(error => {
				console.log(error)
			})
      },
	}
}
</script>