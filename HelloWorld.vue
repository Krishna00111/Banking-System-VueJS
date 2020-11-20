<template>
  <div class="container-fluid" id="main-container">
    <h1 class="text-center text-warning">
      Bank Search
    </h1>

    <div class="container" id="bank-search-container">

      <div class="row my-3">
        <div class="col-sm-6 row">
          <div class="form-group col-sm-6">
            <label for="city">Select City</label>
            <select v-model="city_name" class="form-control" id="city">
              <option>MUMBAI</option>
              <option>PUNE</option>
              <option>NEW DELHI</option>
              <option>BANGALORE</option>
              <option>NAGPUR</option>
              <option>LUCKNOW</option>
            </select>
          </div>

          <div class="col-sm-6 text-center my-4">
            <button class="btn btn-warning" @click="getOrLoadBankData">
              Update Your City
            </button>
          </div>
        </div>

        <div class="col-sm-6">
          <div class="form-group">
            <label for="pagination_size">Number of Results per Page</label>
            <select v-model.number="pages" type="number" class="form-control" id="pagination_size">

              <option>10</option>
              <option>25</option>
              <option>40</option>
              <option>50</option>
              <option>100</option>
            </select>
          </div>
        </div>

      </div>
      <div class="container">

        <div v-if="saved_banks_local && saved_banks_local.length > 0"
             class="container p-2 bg-danger rounded text-white m-3">
          <h3 class="text-center">
            You have {{ saved_banks_local.length }} Favorite Banks.
          </h3>
          <router-link to="/favorite" tag="h5" class="favorite-link">
            View Favorite Banks
          </router-link>
        </div>

        <h3 v-else class="text-center text-warning">
          No banks Favorited Just yet
        </h3>
      </div>
      <div class="container text-center my-4 p-2" id="search_box_container">
        <div class="form-row">
          <div class="col-sm-2">
            <label for="search_text">Search here</label>
          </div>
          <div class="col-sm-8">
            <input type="text" v-model="search_text" class="form-control" id="search_text" placeholder="Search..">
          </div>
        </div>

        <div v-if="results.length > 0" class="container text-center">
          <p v-if="search_text" class="result-text">
            <span class="highlight">{{ number_of_results }}</span>
            search results fetched in <span class="highlight">{{ number_of_pages }}</span> pages using
            search filtering.
          </p>

          <p v-else class="result-text">
            <span class="highlight">{{ number_of_results }}</span>
            search results fetched in <span class="highlight">{{ number_of_pages }}</span> pages,
            you can use pagination to navigate through results or you could narrow down your
            search results using search filtering box.
          </p>
        </div>

        <div class="container-fluid my-5" id="table-container">
          <table class="table table-striped">
            <thead>
            <tr>
              <th>IFSC</th>
              <th>BANK NAME</th>
              <th>CITY</th>
              <th>DISTRICT</th>
              <th>ADDRESS</th>
              <th>FAVORITE</th>
              <th>VIEW DETAILS</th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="each_result in bankResults">
              <td>{{ each_result.ifsc }}</td>
              <td>{{ each_result.bank_name }}</td>
              <td>{{ each_result.city }}</td>
              <td>{{ each_result.district }}</td>
              <td>{{ each_result.address }}</td>
              <td>
                <button v-if="saved_banks_local && saved_banks_local.includes(each_result.ifsc)"
                        class="btn btn-danger m-1" @click="localRemoveBank(each_result.ifsc)">
                  Remove
                </button>

                <button v-else class="btn-success btn m-1 " @click="localSaveBank(each_result.ifsc)">
                  Favorite
                </button>
              </td>
              <td>
                <button class="btn-warning btn" @click="showDetail(each_result)">
                  Details
                </button>
              </td>
            </tr>
            </tbody>
          </table>
          <div v-if="results.length > 0" class="container text-center">
            <h3 class="highlight">You are on page number {{ current_page }}</h3>
          </div>
          <div v-if="results.length > 0" class="container m-2 p-2 text-center">
            <paginate v-if="number_of_pages > 1"
              :page-count="number_of_pages"
              :page-range="3"
              :margin-pages="2"
              :click-handler="nextPage"
              :prev-text="'Prev'"
              :next-text="'Next'"
              :container-class="'pagination'"
              :page-class="'page-item'">
            </paginate>
          </div>
          <div v-else class="container text-center mb-4">
            <ring-loader
              loading="loading" size="120px" color="#000" class="ring">

            </ring-loader>
          </div>
        </div>

      </div>
    </div>


  </div>
</template>

<script>
import RingLoader from 'vue-spinner/src/RingLoader'
import Paginate from 'vuejs-paginate'
import axios from 'axios';
import BankDetailComponent from './BankDetail';

export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      results: [],
      filtered_results: [],
      pages: 10,
      search_text: '',
      city_name: 'MUMBAI',
      current_index: 0,
      saved_banks_local: [],
      number_of_results: null,
      number_of_pages: 1,
      current_page: 1,
      pagination_enabled: false,
      cached_city: null,
      city_not_changed: true
    }
  },
  computed: {
    bankResults() {
      let new_array = [];
      if(this.search_text) {
        this.filtered_results = this.results
          .filter((value) => {
            return (value.address.match(this.search_text.toUpperCase()) ||
              value.bank_name.match(this.search_text.toUpperCase()) ||
              value.ifsc.match(this.search_text.toUpperCase()));
          });
        this.number_of_results = this.filtered_results.length;
        this.number_of_pages = Math.ceil(this.filtered_results.length/this.pages);
        return this.filtered_results.slice(this.current_index, this.current_index + this.pages);
      }
      else {
        this.number_of_results = this.results.length;
        this.number_of_pages = Math.ceil(this.results.length/this.pages);
        return this.results.slice(this.current_index,this.current_index + this.pages);
      }
    },
    isSaved(ifsc_code) {
      if(this.saved_banks_local.includes(ifsc_code))
        return 'Remove';
      else
        return 'Favorite';
    }
  },
  components: {
    BankDetailComponent,
    Paginate,
    RingLoader,
  },
  methods: {
    getOrLoadBankData() {
      this.search_text = '';
      const cached_data = localStorage.getItem('api_data');
      const cached_url = localStorage.getItem('cached_url');
      const cached_city = localStorage.getItem('cached_city');

      if(cached_city && this.city_not_changed)
        this.city_name = cached_city;
      const current_url = 'https://vast-shore-74260.herokuapp.com/banks?city=' + this.city_name;

      // Loading data from the url to be cached for the first time
      if(current_url !== cached_url)
      {
        localStorage.setItem('cached_url', current_url);
        localStorage.setItem('cached_city', this.city_name)
        axios.get(current_url)
          .then((response) => {
            this.results = response.data;
            localStorage.setItem('api_data', JSON.stringify(response.data));
          })
      }
      else {
        const api_Data = localStorage.getItem('api_data');
        this.results = JSON.parse(api_Data);
      }

    },
    loadCachedCity() {
      const saved_city = localStorage.getItem('cached_city');
      this.cached_city = saved_city;
    },
    showDetail(bank) {
      this.$router.push({name: 'detail', params: { passed_bank: bank, ifscCode: bank.ifsc  }});
    },
    nextPage(page_no) {
      this.pagination_enabled = true;
      this.current_page = page_no;
      this.current_index = (page_no-1) * this.pages;
    },
    localSaveBank(bank_ifsc) {
      if(this.saved_banks_local)
      {
        if(!(this.saved_banks_local.includes(bank_ifsc)))
          this.saved_banks_local.push(bank_ifsc);
      }
      else {
        this.saved_banks_local = [];
        this.saved_banks_local.push(bank_ifsc);
        localStorage.setItem('banks', JSON.stringify(this.saved_banks_local));
      }
    },
    localRemoveBank(bank_ifsc) {
      if(this.saved_banks_local)
      {
        let remove_index = this.saved_banks_local.indexOf(bank_ifsc);
        this.saved_banks_local.splice(remove_index, 1);
      }
    },
  },
  watch: {
    // whenever page size changes, this function will run
    pages: function () {
      this.current_index = 0;
      this.current_page = 1;
    },
    // whenever new url is visited and data is updated.
    results: function() {
      this.current_index = 0;
    },
    search_text: function() {
      if(this.pagination_enabled)
      {
        this.current_index = 0;
        this.current_page = 1;
      }
    },
    city_name: function() {
      this.pagination_enabled = false;
      this.current_page = 1;
      this.city_not_changed = false;
    }
  },
  mounted() {
    this.getOrLoadBankData();
    this.loadCachedCity();
    this.saved_banks_local = JSON.parse(localStorage.getItem('banks'));
  },
  updated() {
    localStorage.setItem('banks', JSON.stringify(this.saved_banks_local));
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

  input, select, textarea{
    color: darkcyan !important;
    font-family: "Open Sans", sans-serif;
    font-weight: 800;
  }

  #main-container {
    background-image: url('../images/red.jpg');
    padding-top: 1rem;
  }

  /* Overwriting default CSS bootstrap button classes */
  .btn-success {
    background-color: seagreen !important;
    font-family: "Segoe Print", sans-serif;
  }

  .btn-danger {
    background-color: crimson !important;
    font-family: "Open Sans", sans-serif;
    font-weight: 800;
  }

  .btn-warning {
    background-color: sandybrown !important;
    font-family: Verdana, sans-serif;
    color: lightcyan !important;
    font-weight: 700;

  }

  #table-container {
    width: 100%;
    background-color: mintcream;

    border-radius: 1.5rem;
  }

  table {
    border-collapse:collapse;
    table-layout:fixed;
    width:500px;
  }
  table th {
    color: cornflowerblue;
    font-weight: 800;
    font-family: "Segoe UI Black", sans-serif;
  }
  table td {
    border:solid 2px #4286f4;
    width:200px;
    word-wrap:break-word;
    color: darkslategray;
    font-family: "Open Sans", sans-serif;
    font-weight: 700;
  }

  .detail-link:hover {
    color: darkred;
    cursor: pointer;
  }

  .favorite-link {
    color: lightcyan;
    font-family: "Segoe UI Black";
    font-size: 1.5rem;
    transition: all 0.4s ease-out;
  }

  .favorite-link:hover {
    cursor: pointer;
    color: coral;
    font-size: 2rem;
    transition: all 0.4s ease-in;
  }

  .pagination {
    display: inline-block;
    padding-left: 0;
    margin: 20px 0;
    border-radius: 4px;
  }
  .pagination > li {
    display: inline;
  }
  .pagination > li > a,
  .pagination > li > span {
    position: relative;
    float: left;
    padding: 6px 12px;
    margin: 10px;
    line-height: 1.42857143;
    color: azure;
    text-decoration: none;
    background-color: cornflowerblue;
    border-radius: 1rem;
  }

  .ring {
    margin-left: 45%;
  }

  label {
    color: deepskyblue;
    font-family: "Segoe Print", sans-serif;
    font-size: 1.4rem;
    font-weight: 700;
  }

  .result-text {
    font-size: 1.2rem;
    color: seagreen;
    margin: 30px;
    background-color: beige;
    padding: 1rem;
  }

  .highlight {
    font-family: "Segoe Print", sans-serif;
    font-size: 1.5rem;
    color: darkred;
  }

</style>
