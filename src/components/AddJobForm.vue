<template>
    <div>
      <!-- Button trigger modal -->
      <button type="button" class="btn btn-primary" @click="showModal = true">
        Add Job Offer
      </button>
  
      <!-- Modal -->
      <div v-if="showModal" @close="showModal = false">
        <div>
          <h5>Add Job Offer</h5>
        </div>
        <div>
          <form>
            <div class="form-group">
              <label>Title</label>
              <input type="text" class="form-control" v-model="title" />
            </div>
            <div class="form-group">
              <label>Presentation</label>
              <textarea class="form-control" v-model="presentation" />
            </div>
            <div class="form-group">
              <label>Min Salary</label>
              <input type="number" class="form-control" v-model="min_salary" />
            </div>
            <div class="form-group">
              <label>Max Salary</label>
              <input type="number" class="form-control" v-model="max_salary" />
            </div>
            <div class="form-group">
              <label>Working Place</label>
              <select class="form-control" v-model="working_place">
                <option v-for="item in workingPlaceOptions" :key="item.value" :value="item">
                  {{item.label}}
                </option>
              </select>
            </div>
            <div class="form-group">
              <label>Working Place Country</label>
              <select class="form-control" v-model="working_place_country">
                <option v-for="country in workingPlaceCountryOptions" :key="country.value" :value="country">
                  {{country.label}}
                </option>
              </select>
            </div>
            <div class="form-group">
              <label>Employment Contract Type</label>
              <select class="form-control" v-model="employment_contract_type">
                <option v-for="type in employmentContractTypeOptions" :key="type.value" :value="type">
                  {{type.label}}
                </option>
              </select>
            </div>
            <div class="form-group">
              <label>Contractual Working Time</label>
              <input type="text" class="form-control" v-model="contractual_working_time" />
            </div>
            <div class="form-group">
              <label>Collective Agreement</label>
              <select class="form-control" v-model="collective_agreement">
                <option v-for="agreement in collectiveAgreementOptions" :key="agreement.value" :value="agreement">
                  {{agreement.label}}
                </option>
              </select>
            </div>
            <div class="form-group">
              <label>Flexible Hours</label>
              <select class="form-control" v-model="flexible_hours">
                <option value="true">Yes</option>
                <option value="false">No</option>
              </select>
            </div>
            <div class="form-group">
              <label>Working Hours Modulation System</label>
              <select class="form-control" v-model="working_hours_modulation_system">
                <option value="true">Yes</option>
                <option value="false">No</option>
              </select>
            </div>
          </form>
        </div>
        <div>
          <button type="button" class="btn btn-secondary" @click="showModal = false">
            Close
          </button>
          <button type="button" class="btn btn-primary" @click="addJobOffer">
            Submit
          </button>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios';

  export default {
    data() {
      return {
        showModal: false,
        title: "",
        presentation: "",
        min_salary: 0,
        max_salary: 0,
        working_place: null,
        working_place_country: null,
        employment_contract_type: null,
        contractual_working_time: "",
        collective_agreement: null,
        flexible_hours: false,
        working_hours_modulation_system: false,
        workingPlaceOptions: null,
        workingPlaceCountryOptions: null,
        employmentContractTypeOptions: null,
        collectiveAgreementOptions: null
      };
    },
    mounted() {
        this.fetchFormEnums();
      },
    methods: {
      addJobOffer() {
            const jobOffer = {
                title: this.title,
                presentation: this.presentation,
                min_salary: this.min_salary,
                max_salary: this.max_salary,
                working_place: this.working_place.value,
                working_place_country: this.working_place_country.value,
                employment_contract_type: this.employment_contract_type.value,
                contractual_working_time: this.contractual_working_time,
                collective_agreement: this.collective_agreement.value,
                flexible_hours: this.flexible_hours,
                working_hours_modulation_system: this.working_hours_modulation_system
            };

            axios.post('http://localhost:80/api/jobs', jobOffer, {
				'headers': {
					Authorization: `Bearer ${localStorage.getItem('token')}`,
				}
			})
            .then(() => {
                this.showModal = false;
            })
            .catch(error => {
                alert('Job offer creation failed, please try again');
                console.error(error)
        })
        },
        fetchFormEnums() {
            axios.get('http://localhost:80/api/enums', {
                'headers': {
                    Authorization: `Bearer ${localStorage.getItem('token')}`,
                }
            })
                .then(response => {
                    this.workingPlaceOptions = response.data.working_place,
                    this.workingPlaceCountryOptions = response.data.working_place_country,
                    this.employmentContractTypeOptions = response.data.employment_contract_type,
                    this.collectiveAgreementOptions = response.data.collective_agreement

					this.working_place = this.workingPlaceOptions[0]
					this.working_place_country = this.workingPlaceCountryOptions[0]
					this.employment_contract_type = this.employmentContractTypeOptions[0]
					this.collective_agreement = this.collectiveAgreementOptions[0]
                })
                .catch(error => {
                    console.log(error);
                });
        },
    }
};
  </script>