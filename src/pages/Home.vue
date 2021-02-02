<template>
  <div class="wrapper-content wrapper-content--fixed">
    <section>
      <div class="container">
        <search
          :value="search"
          placeholder="Type username"
          @search="search = $event"
        />
        <button v-if="!repos" class="btn btnPrimary" @click="getRepos">
          Search!
        </button>
        <button v-else class="btn btnPrimary" @click="getRepos">
          Search again!
        </button>
        <div class="error" v-if="error">
          <p>{{ error }}</p>
        </div>

        <div v-if="acc" class="user-card">
          <img :src="acc.avatar_url" alt="" class="user-img" />
          <div class="user-info">
            <p class="user-name">{{ acc.name }}</p>
            <a :href="acc.html_url" target="_blank" class="user-url">{{
              acc.login
            }}</a>
            <p class="user-bio">{{ acc.bio }}</p>
          </div>
        </div>

        <div class="repos__wrapper" v-if="repos">
          <h1 class="repos-title">Repositories</h1>
          <div class="repos-sort">
            <p class="repos-name" @click="sort('name')">Name &#8595;</p>
            <p class="repos-stars" @click="sort('stargazers_count')">
              Stars &#8595;
            </p>
          </div>
          <div class="repos-item" v-for="repo in reposSort" :key="repo.id">
            <div class="repos-info">
              <a class="link" target="_blank" :href="repo.html_url">{{
                repo.name
              }}</a>
              <span>{{ repo.stargazers_count }} ⭐</span>
            </div>
          </div>
          <div class="repos-pagination">
            <button
              :disabled="this.page.current === 1"
              :class="{ disabledBtn: this.page.current === 1 }"
              class="btn btnPrimary"
              @click="prevPage"
            >
              &#8592;
            </button>
            <button class="btn btnPrimary" @click="nextPage">&#8594;</button>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import search from "@/components/Search.vue";
import axios from "axios";

export default {
  components: { search },
  data() {
    return {
      search: "",
      error: null,
      repos: null,
      acc: null,
      currentSort: "name",
      currentSortDir: "asc",
      page: {
        current: 1,
        length: 4,
      },
    };
  },
  computed: {
    reposSort() {
      return this.repos
        .sort((a, b) => {
          let mod = 1;
          if (this.currentSortDir === "desc") mod = -1;
          if (a[this.currentSort] < b[this.currentSort]) return -1 * mod;
          if (a[this.currentSort] > b[this.currentSort]) return 1 * mod;
          return 0;
        })
        .filter((row, index) => {
          let start = (this.page.current - 1) * this.page.length;
          let end = this.page.current * this.page.length;
          if (index >= start && index < end) return true;
        });
    },
  },
  methods: {
    getRepos() {
      axios
        .get(`https://api.github.com/users/${this.search}/repos`)
        .then((res) => {
          console.log(res);
          this.repos = res.data;
          this.error = null;
          this.currentSort = "name";
          this.currentSortDir = "asc";
          this.page.current = 1;
        })
        .catch((err) => {
          console.log(err);
          this.repos = null;
          this.error = `Can't find this user`;
        });
      axios.get(`https://api.github.com/users/${this.search}`).then((acc) => {
        console.log(acc);
        this.acc = acc.data;
      });
    },
    sort(e) {
      if (e === this.currentSort) {
        this.currentSortDir = this.currentSortDir === "asc" ? "desc" : "asc";
      }
      this.currentSort = e;
    },
    // Pagination
    prevPage() {
      if (this.page.current > 1) this.page.current -= 1;
    },
    nextPage() {
      if (this.page.current * this.page.length < this.repos.length)
        this.page.current += 1;
    },
  },
};
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.repos__wrapper {
  width: 400px;
  margin: 30px 0;
}
.repos-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
  padding: 10px 0;
  border-bottom: 1px solid #dbdbdb;
}
.repos-title {
  text-align: center;
  text-transform: uppercase;
}
.repos-sort {
  display: flex;
  justify-content: space-between;
  font-weight: bold;
}
.repos-name,
.repos-stars {
  cursor: pointer;
}
.repos-pagination {
  text-align: center;
  & button {
    margin: 20px;
    .disabled {
      opacity: 0.6;
    }
  }
}
.disabledBtn {
  opacity: 0.6;
}
.user-card {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 400px;
  margin-top: 50px;
  & .user-img {
    width: 100px;
    height: auto;
    border-radius: 50%;
  }
  & .user-info {
    text-align: end;
  }
  & .user-name {
    font-weight: bold;
    font-size: 25px;
  }
}
</style>