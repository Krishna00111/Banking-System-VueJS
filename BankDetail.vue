<template>
  <div class="container-fluid" id="detail-wrapper">
    <div v-if="passed_bank" class="container" id="detail-container">
      <h1 class="text-success text-center">Details for the Bank with IFSC Code
        <span class="ifsc">{{ passed_bank.ifsc }}</span>
      </h1>

      <div class="container">
        <span class="bank-name">{{ passed_bank.bank_name }}</span>
      </div>

      <h3 v-if="favorite_banks && favorite_banks.includes(passed_bank.ifsc)" class="text-center saved-text">
        <span class="important">Important!</span> You have saved this bank in "Favorites"
      </h3>

      <div class="container my-5">
        <div class="row m-2">
          <div class="col-sm-4">
            <h5>Branch ID</h5>
          </div>
          <div class="col-sm-8">
            <h5>{{ passed_bank.bank_id }}</h5>
          </div>
        </div>

        <div class="row m-2">
          <div class="col-sm-4">
            <h5>Branch</h5>
          </div>
          <div class="col-sm-8">
            <h5>{{ passed_bank.branch }}</h5>
          </div>
        </div>

        <div class="row m-2">
          <div class="col-sm-4">
            <h5>Address</h5>
          </div>
          <div class="col-sm-8">
            <h5>{{ passed_bank.address }}</h5>
          </div>
        </div>

        <div class="row m-2">
          <div class="col-sm-4">
            <h5>City</h5>
          </div>
          <div class="col-sm-8">
            <h5>{{ passed_bank.city }}</h5>
          </div>
        </div>

        <div class="row m-2">
          <div class="col-sm-4">
            <h5>District</h5>
          </div>
          <div class="col-sm-8">
            <h5>{{ passed_bank.district }}</h5>
          </div>
        </div>

        <div class="row m-2">
          <div class="col-sm-4">
            <h5>State</h5>
          </div>
          <div class="col-sm-8">
            <h5>{{ passed_bank.state }}</h5>
          </div>
        </div>
      </div>
    </div>

    <h3 class="text-danger text-center" v-else>You're seeing this message because you reloaded a details
      page which receives props which are stored in the local storage. Please go back to the main page again.
      Access this page from there.</h3>
  </div>


</template>

<script>
  export default {
    name: 'BankDetail',
    props: {
      passed_bank: {
        type: Object
      }
    },
    data() {
      return {
        favorite_banks: null
      }
    },
    methods: {

    },
    computed: {

    },
    created() {
      this.favorite_banks = localStorage.getItem('banks');
      this.favorite_banks = JSON.parse(this.favorite_banks);

    }
  }
</script>

<style scoped>
  #detail-wrapper {
    background-image: url('../images/red.jpg');
    padding-top: 1rem;
    padding-bottom: 1rem;
  }
  .ifsc {
    font-family: "Segoe UI Semibold", sans-serif;
    font-size: 2rem;
    color: crimson;
    margin: 1.5rem;
  }

  .row {
    margin-top: 2rem;
    margin-bottom: 2rem;
  }

  #detail-container {
    background-color: lemonchiffon;
    height: auto;
    width: 80%;
    margin: auto;
    padding: 1.5rem;
    border-radius: 2rem;
  }

  .bank-name {
    font-size: 3rem;
    margin: 2rem;
    font-family: 'Open Sans', sans-serif;
    color: teal;
  }

  .important {
    color: indianred;
    font-size: 2rem;
    font-family: "Segoe UI Black", sans-serif;
  }

  .saved-text {
    color: seagreen;
    background-color: mintcream;
    border-radius: 1.3rem;
    margin: 1rem;
    padding: 1rem;
  }

  h5 {
    font-size: 1.5rem;
    color: darkolivegreen;
  }
</style>
