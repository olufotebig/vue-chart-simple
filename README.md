# vue-chart-simple

> A graph project with Vue.js and Chart.js

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```

REQUIREMENTS
* Each data point needs to be hoverable
* There has to be 2 possible views for the whole graph - one showing WEEKLY view & one showing MONTHLY view
* The date has to be converted to the following format:
	* Day of week for WEEKLY view
	* DD/MM for MONTHLY view
* A button to filter the graph for weekly and monthly performance
* Display total sales for the selected period
* Use Vue.js, Laravel and Chart.js