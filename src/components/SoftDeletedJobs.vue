<template>
    <div>
    <h2>Jobs supprimés</h2>
      <div v-for="job in softDeletedJobs" :key="job.id">
        <h3>{{ job.title }}</h3>
        <button @click="restoreJob(job.id)">Restore</button>
      </div>
    </div>
  </template>

  <script>
  import axios from 'axios';

  export default {
    props: ['softDeletedJobs'],    
    methods: {
      restoreJob(id) {
        axios.put(`http://localhost:80/api/jobs/${id}/restore`, {}, {
                'headers': {
                    Authorization: `Bearer ${localStorage.getItem('token')}`,
                }
            })
          .then(() => {
            this.$emit('restoredJobOffer')
          })
          .catch(error => {
            console.log(error)
          })
      },
    }
  }
  </script>