<script>

  import { Line } from 'vue-chartjs'

  export default {
    extends: Line,
    props:['line_data'],
    mounted() {
      this.renderLineChart();
    },
    computed: {
      // chartData: function() {
      //   return this.line_data;
      // }
      labelData: function() {
        return this.line_data.map(x => x.label);
      },
      valueData: function() {
        return this.line_data.map(x => x.value);
      }
    },
    methods: {
      renderLineChart: function() {
        this.renderChart(
          {
            labels: this.labelData,
            datasets: [
              {
                label: "Data One",
                backgroundColor: "#f87979",
                data: this.valueData
              }
            ]
          },
          { responsive: true, maintainAspectRatio: false }
        );
      }
    },
    watch: {
      data: function() {
        this._chart.destroy();
        this.renderLineChart();
      }
    }
  }

</script>
