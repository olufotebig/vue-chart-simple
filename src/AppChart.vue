<script>
import { Line, mixins } from "vue-chartjs";
const { reactiveProp } = mixins;

export default {
  extends: Line,
  mixins: [reactiveProp],
  props: [],
  mounted() {
    this.renderChart(
      // this.chartData is created in the mixin.
      // If you want to pass options please create a local options object
      this.chartData,
      {
        responsive: true,
        maintainAspectRatio: true,
        scales: { yAxes: [{ ticks: { beginAtZero: true } }] }
      }
    );
  }
};
</script>
