<template>
<div>

  <a-input-search placeholder="input search text" style="width: 200px" @search="onSearch" />

  <a-row>
  <a-spin size="large" v-if="searching" class="center" style="margin-left:25%;margin-top:15%" tip="Loading..."/>
  </a-row>
  <div  v-if="!searching" class="center">
  <apexchart
      type="bar"
      :options="options"
      :series="series"
  ></apexchart>
  </div>
  <a-drawer
      title="Basic Drawer"
      placement="bottom"
      :visible="true"
      :mask="false"

  >
    <p>Some contents...</p>
    <p>Some contents...</p>
    <p>Some contents...</p>
  </a-drawer>
</div>
</template>

<script>
export default {
  name: 'search',
  data() {
    return {
      searchQuery: "",
      searching: false,
      publishYears: {},
      chartYears: [],
      chartData: [],
      options: {
        chart: {
          id: "bookChart"
        },
        xaxis: {
          categories: [],
          type: 'category'
        }
      },
      series: [
        {
          name: "Book count",
          data: []
        }
      ]
      }
    },
  methods: {
    onSearch(val) {
      this.searchQuery = val;
      this.fetch();
    },
    fetch () {
      const axios = require('axios');
      let me = this;
      me.searching = true;
      axios({
        method: 'get',
        url: 'http://openlibrary.org/search.json',
        responseType: 'json',
        params: {
          q: me.searchQuery
        }
      })
          .then(function (response) {
            me.searching = false;
            //console.log(response)
            let docs = response.data && response.data.docs;
            for(let doc of docs){
              let pubYear = doc.first_publish_year;
                if(!me.publishYears[pubYear]) {
                  me.publishYears[pubYear] = 1;
                } else {
                  me.publishYears[pubYear]++;
                }
              }
            for(const key in me.publishYears){
              me.chartYears.push(key);
              me.chartData.push(me.publishYears[key])
            }
            me.options.xaxis.categories = me.chartYears;
            me.series[0].data = me.chartData;
            console.log(me.publishYears)
            console.log(me.chartYears);
            console.log(me.chartData);
          })
      .catch( function(err){
        me.searching = false;
        console.log(err);
      });
    }
  }
}
</script>
<style>
.center {
  margin: auto;
  width: 50%;
}
</style>
