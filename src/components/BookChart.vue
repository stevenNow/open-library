<template>
  <div></div>
</template>
<script>
export default {
  name: "book-chart",
  data() {
    return {
      options: {
        chart: {
          id: "bookChart",
        },
        xaxis: {
          categories: [1, 2, 3],
        },
      },
      series: [
        {
          name: "series-1",
          data: [1, 2, 3],
        },
      ],
    };
  },
};
</script>
