<template>
  <div id="app">
    <h1>{{ title }}</h1>
    <div>
      <label for=""><input type="radio" name="graphview" value="week" v-model="activeView"> Week</label>
      <label for=""><input type="radio" name="graphview" value="month" v-model="activeView"> Month</label>
    </div>
    <div>
      <select  v-model="selectYear">
        <option v-for="item in Object.keys(parsedData)" :key="item" :value="item">{{ item }}</option>
      </select>

      <select  v-model="selectPeriod">
        <option v-for="item in periodOptions" :key="item.key" :value="item.key" :disabled="disabledPeriod(item.key)">{{ item.shortName }}</option>
      </select>
    </div>
    <div v-if="Object.keys(parsedData).length">
      <p>Total sales: {{ `$${periodSalesSum}` }}</p>
      <div style="min-width: 320px; max-width: 600px;  margin: 24px auto;">
        <app-chart :chart-data="plotData"/>
      </div>
    </div>
    <div v-else>
      <p>Loading graph data</p>
    </div>
  </div>
</template>

<script>
const moment = require("moment");
import AppChart from "./AppChart.vue";
export default {
  name: "app",
  components: {
    AppChart
  },
  created() {
    let dataUrl = "https://private-627474-nulisec.apiary-mock.com/sales";
    const that = this;
    const request = new XMLHttpRequest();
    request.open("GET", dataUrl, true);
    
    request.onload = function() {
      if (request.status >= 200 && request.status < 400) {
        // Success!
        var resp = JSON.parse(request.responseText);
        that.updateparsedData(resp);
      } else {
        // We reached our target server, but it returned an error
      }
    };

    request.onerror = function() {
      // There was a connection error of some sort
    };

    request.send();
  },
  computed: {
    periodOptions() {
      if (this.activeView === "month") {
        return this.optimizer.months;
      }
      if (this.activeView === "week") {
        const options = [];
        for (let i = 1; i < 53; i++) {
          options.push({
            key: `${i > 9 ? "" + i : "0" + i}`,
            shortName: `Week-${i > 9 ? "" + i : "0" + i}`
          });
        }
        return options;
      }
    },
    plotData() {
      if (this.selectYear === "" || this.selectPeriod == "") {
        return;
      }

      if (this.activeView === "month") {
        let periodLabels = [];
        let periodData = [];
        let salesSum = 0;

        const lastDay = this.optimizer.max_last_days[
          parseInt(this.selectPeriod) - 1
        ];
        for (let dd = 1; dd <= lastDay; dd++) {
          periodLabels.push(
            `${dd > 9 ? "" + dd : "0" + dd}/${this.selectPeriod}`
          );
          periodData.push(0);
        }

        let dataSource = this.parsedData[this.selectYear][this.selectPeriod]; //TODO: catch if selectPeriod is invalid
        for (var key in dataSource) {
          if (dataSource.hasOwnProperty(key)) {
            periodData[parseInt(key) - 1] = dataSource[key];
            salesSum += dataSource[key];
          }
        }
        this.periodSalesSum = salesSum; // TODO: improve
        return {
          labels: periodLabels,
          datasets: [
            {
              label: "Sales",
              data: periodData,
              fill: false,
              borderColor: "rgb(0, 12, 192)",
              lineTension: 0.1
            }
          ]
        };
      }
      if (this.activeView === "week") {
        let periodLabels = this.optimizer.labels_week;
        let periodData = [];
        let salesSum = 0;
        const weekDay = moment(`${this.selectYear}W${this.selectPeriod}`);

        let month;
        let date;
        for (let i = 0; i < 7; i++) {
          month = weekDay.month() + 1; // moment month is zero indexed so make one indexed
          month = month > 9 ? "" + month : "0" + month; // I have used this a number of times, extract to function
          let dataSource = this.parsedData[this.selectYear][month];
          if (dataSource === undefined) {
            periodData.push(0);
          } else {
            date =
              weekDay.date() > 9 ? "" + weekDay.date() : "0" + weekDay.date();
            if (dataSource[date] === undefined) {
              periodData.push(0);
            } else {
              periodData.push(dataSource[date]);
              salesSum += dataSource[date];
            }
          }
          weekDay.add(1, "d");
        }
        this.periodSalesSum = salesSum; // TODO: improve
        return {
          labels: periodLabels,
          datasets: [
            {
              label: "Sales",
              data: periodData,
              fill: false,
              borderColor: "rgb(0, 12, 192)",
              lineTension: 0.1
            }
          ]
        };
      }
    }
  },
  data() {
    return {
      activeView: "week",
      selectYear: "",
      selectPeriod: "",
      optimizer: {
        max_last_days: [31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31], // TODO: Feb sould be 28 or 29
        labels_week: [
          "Monday",
          "Tuesday",
          "Wednesday",
          "Thursday",
          "Friday",
          "Saturday",
          "Sunday"
        ],
        months: [
          {
            key: "01",
            shortName: "Jan"
          },
          {
            key: "02",
            shortName: "Feb"
          },
          {
            key: "03",
            shortName: "Mar"
          },
          {
            key: "04",
            shortName: "Apr"
          },
          {
            key: "05",
            shortName: "May"
          },
          {
            key: "06",
            shortName: "Jun"
          },
          {
            key: "07",
            shortName: "Jul"
          },
          {
            key: "08",
            shortName: "Aug"
          },
          {
            key: "09",
            shortName: "Sep"
          },
          {
            key: "10",
            shortName: "Oct"
          },
          {
            key: "11",
            shortName: "Nov"
          },
          {
            key: "12",
            shortName: "Dec"
          }
        ]
      },
      parsedData: {}, // TODO: describe structure of object in comments or docs
      periodSalesSum: 0,
      title: "Vue.js and Chart.js Simple App"
    };
  },
  methods: {
    updateparsedData(apiData) {
      const newData = {};
      apiData.forEach(function(element) {
        const split_date = element.published_at.substr(0, 10).split("-");
        // TODO: can the if statements be reduced?
        if (newData[split_date[0]] === undefined) {
          newData[split_date[0]] = {};
        }
        if (newData[split_date[0]][split_date[1]] === undefined) {
          newData[split_date[0]][split_date[1]] = {};
        }
        if (
          newData[split_date[0]][split_date[1]][split_date[2]] === undefined
        ) {
          newData[split_date[0]][split_date[1]][split_date[2]] = 0; // what if sales val is float?
        }
        newData[split_date[0]][split_date[1]][split_date[2]] += parseInt(
          element.sale.substr(1)
        );
      });

      this.parsedData = newData;
    },
    disabledPeriod(key) {
      // TODO: notify user why month is disabled. e.g select a year
      if (this.parsedData[this.selectYear] === undefined) return true;
      if (this.activeView == "month") {
        return this.parsedData[this.selectYear][key] === undefined;
      }
      return false;
    }
  }
};
</script>

<style lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 20px;
}

h1,
h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

// TODO: Style disabled options in month select: Maybe use icons
</style>
