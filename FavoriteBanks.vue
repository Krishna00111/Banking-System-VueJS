<template>
  <div class="container-fluid" id="favorite-container">
    <h1 class="text-center text-white p-2">
      List of Favorited Banks with their IFSC Codes
    </h1>
    <h3 v-if="delete_msg" class="text-white">
      {{ delete_msg }}
    </h3>
    <h3 v-else v-for="item in favorite_banks" class="text-center favorite-text my-3">
      {{ item }}
    </h3>

    <div v-if="!delete_msg" class="container text-center">
      <button class="btn btn-danger" @click="deleteFavorites">
        DELETE FAVORITES
      </button>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'FavoriteBanks',
    data() {
      return {
        favorite_banks: '',
        delete_msg: ''
      }
    },
    methods: {
      deleteFavorites() {
        localStorage.removeItem('banks');
        this.delete_msg = 'You favorites list was successfully cleared!';
      }
    },
    created() {
      this.favorite_banks = localStorage.getItem('banks');
      this.favorite_banks = JSON.parse(this.favorite_banks);

    }
  }
</script>

<style scoped>
  #favorite-container {
    background-image: url('../images/red.jpg');
    padding-top: 1rem;
    padding-bottom: 1rem;
  }

  .favorite-text {
    color: lightcyan;
    font-family: "Open Sans", sans-serif;
    font-weight: 700;
  }
</style>
