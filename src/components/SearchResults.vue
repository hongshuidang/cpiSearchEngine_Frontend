<template>
  <div>
    <div class="py-1 mb-2">
      <b-input-group>
        <b-input v-model="searchQuery" />
        <b-btn slot="append" variant="primary" @click="search1()"><i class="ion ion-ios-search"></i>&nbsp; Search</b-btn>
      </b-input-group>
    </div>

    <div v-if="amount > 0">
      <span>About {{amount}} results ({{timeCost}} nanoseconds)</span>
      <b-card no-body>
          <b-list-group :flush="true">
              <li v-for="page in pages" :key="page.title + page.text + page.url" class="list-group-item py-4">
                  <a :href="page.url" class="text-big">{{page.title}}</a>
                  <div v-if="page.url" class="small mt-1">
                      <a :href="page.url" class="text-success">{{page.url}}</a>
                      <span>&nbsp; <span class="text-muted">·</span> &nbsp;{{page.freq}} found</span>
                  </div>
                  <div v-if="page.text" class="mt-2">{{page.text}}</div>
              </li>
          </b-list-group>
      </b-card>
      <b-pagination :total-rows="amount" v-model="currentPage" :per-page="5"  @input="search2" class="mt-3" />
    </div>
    <div v-else>
      <div v-if="suggestions && suggestions.length > 0">
          <div>
            <span>Did you mean:  </span>
            <span v-for="suggestion in suggestions">
              <button @click="searchRecommendedWord(suggestion)" class="d-inline-block mr-2" style="border-color: transparent; background: #26B4FF;  color: #fff;">{{suggestion}}</button>
            </span>
          </div>

          <p style="padding-top:.33em">Your search - <font color="red">{{searchQuery}}</font> - did not match any webpage.</p>
          <p style="padding-top:.33em">Suggestions: </p>
          <ul>
            <li>Click on one recommended word, whose edit distance is 1 or 2 from <font color="red">{{searchQuery}}</font></li>
            <li>Make sure that your word is spelled correctly</li>
          </ul>
      </div>
      <div v-else-if="suggestions === null">
        <p style="padding-top:.33em">Your search - <font color="red">{{searchQuery}}</font> - did not match any webpage.</p>
        <p style="padding-top:.33em">Make sure that your word is spelled correctly</p>
      </div>
    </div>
  </div>
</template>


<style src="@/vendor/styles/pages/search.scss" lang="scss"></style>
<script>
import vSelect from 'vue-select'
export default {
  name: 'search-results',
  metaInfo: {
    title: 'Search results'
  },
  components: {
    vSelect,
  },
  data: () => ({
    curTab: 'pages',
    searchQuery: '',
    pages: [],
    amount: 0,
    timeCost: 0,
    suggestions: [],
    currentPage: 1,
    options: [],
  }),
  methods: {
    search1 () {
      this.currentPage = 1;
      let url = this.$store.state.dataUrl + "/cpi/ranktest?word="+this.searchQuery+"&stringPageOffset="+this.currentPage;
      // let url = "http://localhost:2020/cpi/ranktest?word="+this.searchQuery+"&stringPageOffset="+this.currentPage;
      console.log(url);

      this.$http.get(url).then(response => {
        let result = response.data

        this.amount = result.n;
        this.timeCost = result.time;
        this.pages = result.pages;
        this.suggestions = result.suggestions;

        console.log(result);
        this.$showNotification('acNotification', 'success', 'Search', 'Search result on ' + this.searchQuery + ' is returned back successfully!');
      }, error => {
        this.$showNotification('acNotification', 'error', 'Domain-settings', 'Error occurres when searching ' + this.searchQuery + '!');
      });
    },

    search2 () {
      let url = this.$store.state.dataUrl + "/cpi/ranktest?word="+this.searchQuery+"&stringPageOffset="+this.currentPage;
      console.log(url);

      this.$http.get(url).then(response => {
        let result = response.data

        this.amount = result.n;
        this.timeCost = result.time;
        this.pages = result.pages;
        this.suggestions = result.suggestions;

        console.log(result);
        this.$showNotification('acNotification', 'success', 'Search', 'Search result on ' + this.searchQuery + ' is returned back successfully!');
      }, error => {
        this.$showNotification('acNotification', 'error', 'Domain-settings', 'Error occurres when searching ' + this.searchQuery + '!');
      });
    },

    searchRecommendedWord(suggestion) {
      console.log(suggestion);
      this.searchQuery = suggestion;
      this.search1();
    }
  }
}
</script>
