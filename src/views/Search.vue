<template>
  <div>
    <a-input-search
      placeholder="input search text"
      style="width: 200px"
      @search="onSearch"
    />

    <a-row>
      <a-spin
        size="large"
        v-if="searching"
        class="center"
        style="margin-left: 25%; margin-top: 15%"
        tip="Loading..."
      />
    </a-row>
    <div v-if="!searching" class="center">
      <apexchart
        type="bar"
        :options="options"
        :series="series"
        @dataPointMouseEnter="hoverPoint"
      ></apexchart>
    </div>
    <a-drawer
      :title="drawerTitle"
      placement="bottom"
      :visible="drawerVisible"
      :mask="false"
      :closable="true"
      @close="closeDrawer"
      height="400"
    >
      <a-row>
        <a-col span="4" v-for="i in images" :key="i">
          <a target="_blank" rel="noopener noreferrer" :href="i.bookUrl">
            <a-image
              style="margin-top: 25px"
              :width="200"
              :src="i.imgUrl"
              :preview="false"
              :placeholder="true"
            />
          </a>
        </a-col>
      </a-row>
    </a-drawer>
  </div>
</template>

<script>
export default {
  name: "search",
  data() {
    return {
      searchQuery: "",
      searching: false,
      publishYears: {},
      covers: {},
      images: [],
      chartYears: [],
      chartData: [],
      drawerVisible: false,
      drawerTitle: "Books",
      options: {
        chart: {
          id: "bookChart",
        },
        xaxis: {
          categories: [],
          type: "category",
        },
      },
      series: [
        {
          name: "Book count",
          data: [],
        },
      ],
    };
  },
  methods: {
    onSearch(val) {
      this.searchQuery = val;
      this.fetch();
    },
    closeDrawer() {
      this.drawerVisible = false;
    },
    hoverPoint(event, chartContext, config) {
      this.drawerVisible = true;
      this.images = [];
      let index = config.dataPointIndex;
      let year = this.options.xaxis.categories[index];
      let coverIds = this.covers[year];
      for (let c of coverIds) {
        if (c) {
          this.images.push({
            imgUrl: "http://covers.openlibrary.org/b/id/" + c.imgId + ".jpg",
            bookUrl: "https://openlibrary.org/" + c.workLink,
          });
        }
      }
      this.drawerTitle = "Books from " + year;
    },
    fetch() {
      const axios = require("axios");
      let me = this;
      me.searching = true;
      axios({
        method: "get",
        url: "http://openlibrary.org/search.json",
        responseType: "json",
        params: {
          q: me.searchQuery,
        },
      })
        .then(function (response) {
          me.searching = false;
          console.log(response);
          let docs = response.data && response.data.docs;
          for (let doc of docs) {
            let pubYear = doc.first_publish_year;
            if (!me.publishYears[pubYear]) {
              me.publishYears[pubYear] = 1;
            } else {
              me.publishYears[pubYear]++;
            }
            let coverImage = doc.cover_i;
            let workLink = doc.key;
            if (!me.covers[pubYear]) {
              me.covers[pubYear] = [{ imgId: coverImage, workLink: workLink }];
            } else {
              me.covers[pubYear].push({
                imgId: coverImage,
                workLink: workLink,
              });
            }
          }
          console.log("covers");
          console.log(me.covers);
          for (const key in me.publishYears) {
            if (key !== "undefined") {
              me.chartYears.push(key);
              me.chartData.push(me.publishYears[key]);
            }
          }
          me.options.xaxis.categories = me.chartYears;
          me.series[0].data = me.chartData;
          console.log(me.publishYears);
          console.log(me.chartYears);
          console.log(me.chartData);
        })
        .catch(function (err) {
          me.searching = false;
          console.log(err);
        });
    },
  },
};
</script>
<style>
.center {
  margin: auto;
  width: 50%;
}
</style>
