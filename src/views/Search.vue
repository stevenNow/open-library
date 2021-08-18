<template>
  <div>
    <a-input-search
      placeholder="Search"
      style="width: 200px"
      @search="onSearch"
    />
    <a-button
      v-if="pageCount > 1"
      style="margin-left: 25px"
      @click="loadNextPage"
      :disabled="currentPage >= pageCount"
    >
      More
    </a-button>
    <p>Displaying {{ displayCount }} of {{ resultCount }} results.</p>
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
        @dataPointSelection="clickPoint"
      ></apexchart>
    </div>
    <a-drawer
      :title="drawerTitle"
      placement="bottom"
      :visible="drawerVisible"
      :mask="false"
      :closable="true"
      @close="closeDrawer"
      height="350"
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
import axios from "axios";

export default {
  name: "search",
  data() {
    return {
      searchQuery: "",
      resultCount: 0,
      displayCount: 0,
      pageCount: 0,
      currentPage: 1,
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
    clickPoint(event, chartContext, config) {
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
      this.setDrawerTitle(year);
    },
    setDrawerTitle(year) {
      this.drawerTitle = "Books from " + year;
    },
    setResultCount(resultCount) {
      this.resultCount = resultCount;
    },
    setPageCount(resultCount) {
      this.pageCount = Math.ceil(resultCount / 100);
    },
    resetPagination() {
      this.pageCount = 0;
      this.displayCount = 0;
      this.resultCount = 0;
      this.currentPage = 1;
    },
    setDisplayCount(displayed) {
      this.displayCount += displayed;
    },
    loadNextPage() {
      let me = this;
      me.searching = true;
      me.currentPage++;
      axios({
        method: "get",
        url: "http://openlibrary.org/search.json",
        responseType: "json",
        params: {
          q: me.searchQuery,
          page: me.currentPage,
        },
      })
        .then(function (response) {
          if (!response.data.error) {
            let docs = response.data && response.data.docs;
            me.setDisplayCount(docs.length);
            me.setData(docs, me);
          }
          me.searching = false;
        })
        .catch(function (err) {
          me.searching = false;
          console.log(err);
        });
    },
    fetch() {
      let me = this;
      me.searching = true;
      me.resetPagination();
      me.publishYears = {};
      me.covers = {};
      me.drawerVisible = false;
      axios({
        method: "get",
        url: "http://openlibrary.org/search.json",
        responseType: "json",
        params: {
          q: me.searchQuery,
        },
      })
        .then(function (response) {
          let resultCount = response.data && response.data.num_found;
          let docs = response.data && response.data.docs;
          me.setPageCount(resultCount);
          me.setResultCount(resultCount);
          me.setDisplayCount(docs.length);
          me.setData(docs, me);
          me.searching = false;
        })
        .catch(function (err) {
          me.searching = false;
          console.log(err);
        });
    },
    setData(docs, me) {
      for (let doc of docs) {
        let pubYear = doc.first_publish_year;
        me.setPubYear(pubYear, me);
        me.setCover(pubYear, doc, me);
      }
      me.resetChart(me);
      me.setChartData(me);
    },
    setPubYear(pubYear, me) {
      if (!me.publishYears[pubYear]) {
        me.publishYears[pubYear] = 1;
      } else {
        me.publishYears[pubYear]++;
      }
    },
    setCover(pubYear, doc, me) {
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
    },
    resetChart(me) {
      me.options.xaxis.categories = [];
      me.series[0].data = [];
    },
    setChartData(me) {
      for (const key in me.publishYears) {
        if (key !== "undefined") {
          me.options.xaxis.categories.push(key);
          me.series[0].data.push(me.publishYears[key]);
        }
      }
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
