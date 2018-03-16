<template>
  <div id="app">
    <search-list @selectCity="selectCity" :searchList="searchList"></search-list>
  </div>
</template>

<script>
  import SearchList from './components/SearchList'
  import axios from 'axios'

  export default {
    name: 'App',
    data() {
      return {
        searchList: {}
      }
    },
    created() {
      axios.get('/api/search-list')
        .then((res) => {
          this.searchList = res.data.searchList;
        })
    },
    methods: {
      selectCity(item) {
        console.log("点击的是：", item.cityName);

      }
    },
    components: {
      SearchList
    }
  }
</script>

<style>
  body {
    margin: 0;
  }

  h3 {
    margin: 0;
  }

  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  li {
    margin: 0;
    list-style: none;
  }

  #app {

  }
</style>
