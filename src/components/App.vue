<template>
  <div id="app">
    <p class="control has-addons has-addons-centered">
      <input class="input" type="text" placeholder="GitHub account" v-model="name">
      <input class="input" type="text" placeholder="limit" v-model="limit">
      <input class="input" type="text" placeholder="thresh" v-model="thresh">
      <a class="button is-primary" @click="fetchUser">
        Search
      </a>
    </p>
    <div class="data has-text-centered">
      <p class="total" v-if="vTotal">total: <span class="total-number" v-text="vTotal"></span>  ⭐</p>
      <ul class="rank has-text-left">
        <li v-for="item in vRepos" v-if="vRepos"><span class="name">{{item.name}}</span>{{item.star}}  ⭐</li>
      </ul>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  const fixRate = '?client_id=60d16794dabd9b52de64&client_secret=5a60e7ab2dd5135ab1cf28bf43482e1f3d15b8a5';

  export default {
    name: 'app',
    data() {
      return {
        name: '',
        limit: 10,
        thresh: 1,
        total: 0,
        vTotal: null,
        repos: [],
        vRepos: []
      }
    },
    methods: {
      fetchUser() {
        this.repos = []
        this.vRepos = []
        this.vTotal = null;
        this.total = 0
        if (!this.name) {
          return
        }

        axios.get('https://api.github.com/users/' + this.name + fixRate)
        .then((response) => {
          var totalRepos = response.data.public_repos
          var pages = Math.ceil( totalRepos / 100)
          while(pages--) {
            this.fetchRepos(pages + 1)
          }
        })
        .catch((error) => {
          console.log(error)
        });
      },
      fetchRepos(page) {
        axios.get('https://api.github.com/users/' + this.name + '/repos' + fixRate + '&per_page=100&page=' + page)
        .then((response) => {
          this.saveReposData(response.data)
          if (page === 1) {
            this.filter(this.repos)
            this.vTotal = this.total
          }
        })
        .catch((error) => {
          console.log(error);
        });
      },
      saveReposData(repos) {
        repos.forEach((repo) => {
          this.total += repo.stargazers_count
          this.repos.push({
            name: repo.name,
            star: repo.stargazers_count
          })
        })
      },
      filter(arr) {
        var newArr = arr.filter((value) => {
          return value.star >= this.thresh
        })

        newArr.sort((a, b) => {
          return b.star - a.star
        })

        this.vRepos = newArr.slice(0, this.limit)
      }
    },
  }
</script>

<style lang="scss">
  html, body, #app {
    height: 100%;
  }
  body {
    background-color: #f9f9f9;
    margin: 0;
  }
  * {
    box-sizing: border-box;
  }
  .control {
    padding-top: 50px;
  }
  .rank {
    max-width: 285px;
    margin: 0 auto;
    li {
      margin: 10px 0;
    }
    .name {
      display: inline-block;
      min-width: 70%;
    }
  }
</style>