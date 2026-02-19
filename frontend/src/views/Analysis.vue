<template>
  <v-container fluid class="bg-surface">
    <v-row justify="center">
      <v-col cols="12" md="10">
        <v-row>
          <v-col cols="12" md="4">
            <v-sheet class="pa-4" rounded="lg" border>
              <p class="text-h6 mb-4">Enter date range for Analysis</p>
              <v-text-field v-model="start" label="Start Date" type="date" density="compact" variant="solo-inverted" flat></v-text-field>
              <v-text-field v-model="end" label="End Date" type="date" density="compact" variant="solo-inverted" flat></v-text-field>
              <v-btn color="primary" variant="tonal" block @click="updateCards(); updateLineCharts();updateHistogramCharts();">Analyze</v-btn>
            </v-sheet>
          </v-col>

          <v-col cols="12" md="4">
            <v-card variant="outlined" color="primary" rounded="lg" class="pa-2">
              <v-card-title>Temperature</v-card-title>
              <v-chip-group class="justify-center">
                <v-tooltip text="Min" location="bottom">
                  
                    <v-chip  color="primaryContainer" variant="flat">{{ temperature.min }}</v-chip>
                  
                </v-tooltip>
                <v-tooltip text="Max" location="bottom">
                  
                    <v-chip  color="primaryContainer" variant="flat">{{ temperature.max }}</v-chip>
                  
                </v-tooltip>
                <v-tooltip text="Range" location="bottom">
                  
                    <v-chip  color="primaryContainer" variant="flat">{{ temperature.range }}</v-chip>
                  
                </v-tooltip>
              </v-chip-group>
              <v-card-text class="text-center">
                <span class="text-h1 text-primary font-weight-bold">{{ temperature.avg }}</span>
              </v-card-text>
            </v-card>
          </v-col>

          <v-col cols="12" md="4">
            <v-card variant="outlined" color="primary" rounded="lg" class="pa-2">
              <v-card-title>Humidity</v-card-title>
              <v-chip-group class="justify-center">
                <v-tooltip text="Min" location="bottom">
                  
                    <v-chip  color="primaryContainer" variant="flat">{{ humidity.min }}</v-chip>
                  
                </v-tooltip>
                <v-tooltip text="Max" location="bottom">
                  
                    <v-chip  color="primaryContainer" variant="flat">{{ humidity.max }}</v-chip>
                 
                </v-tooltip>
                <v-tooltip text="Range" location="bottom">
                  
                    <v-chip  color="primaryContainer" variant="flat">{{ humidity.range }}</v-chip>
                  
                </v-tooltip>
              </v-chip-group>
              <v-card-text class="text-center">
                <span class="text-h1 text-primary font-weight-bold">{{ humidity.avg }}</span>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <v-row class="mt-5">
          <v-col cols="12"><div id="container"></div></v-col>
          <v-col cols="12"><div id="container0"></div></v-col>
          <v-col cols="12"><div id="container1"></div></v-col>
          <v-col cols="12" md="6"><div id="container2"></div></v-col>
          <v-col cols="12" md="6"><div id="container3"></div></v-col>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, reactive, onMounted } from "vue";
import Highcharts from 'highcharts';

// VARIABLES
const start = ref("");
const end = ref("");
const temperature = reactive({ min: 0, max: 0, avg: 0, range: 0 });
const humidity = reactive({ min: 0, max: 0, avg: 0, range: 0 });

// Chart refs to update data later
const charts = reactive({});

const updateCards = async () => {
  if (!!start.value && !!end.value) {
    let startDate = new Date(start.value).getTime() / 1000;
    let endDate = new Date(end.value).getTime() / 1000;
    
    // Assuming AppStore is globally available or imported
    const temp = await AppStore.getTemperatureMMAR(startDate, endDate);
    const humid = await AppStore.getHumidityMMAR(startDate, endDate);

    temperature.max = temp[0].max.toFixed(1);
    temperature.min = temp[0].min.toFixed(1);
    temperature.avg = temp[0].avg.toFixed(1);
    temperature.range = (temp[0].max - temp[0].min).toFixed(1);

    humidity.max = humid[0].max.toFixed(1);
    humidity.min = humid[0].min.toFixed(1);
    humidity.avg = humid[0].avg.toFixed(1);
    humidity.range = (humid[0].max - humid[0].min).toFixed(1);
  }
};

const updateLineCharts = async ()=>{
if(!!start.value && !!end.value){
    // Convert output from Textfield components to 10 digit timestamps
    let startDate = new Date(start.value).getTime() / 1000;
    let endDate = new Date(end.value).getTime() / 1000;
    // Fetch data from backend
    const data = await AppStore.getAllInRange(startDate,endDate);
    // Create arrays for each plot
    let temperature = [];
    let heatindex = [];
    let humidity = [];
    // Iterate through data variable and transform object to format recognized by highcharts
    data.forEach(row => {
    temperature.push({"x": row.timestamp * 1000, "y": parseFloat(row.temperature.toFixed(2)) });
    heatindex.push({ "x": row.timestamp * 1000,"y": parseFloat(row.heatindex.toFixed(2)) });
    humidity.push({ "x": row.timestamp * 1000,"y": parseFloat(row.humidity.toFixed(2)) });
    });
    // Add data to Temperature and Heat Index chart
    tempChart.value.series[0].setData(temperature);
    tempChart.value.series[1].setData(heatindex);
    tempChart.value.series[2].setData(humidity);
    }
    };

const updateHistogramCharts = async () =>{
// Retrieve Min, Max, Avg, Spread/Range for Column graph
if(!!start.value && !!end.value){
// 1. Convert start and end dates collected fron TextFields to 10 digit timestamps
const startDate = new Date(start.value).getTime() / 1000;
const endDate = new Date(end.value).getTime() / 1000;
// Subsequently, create startDate and endDate variables and then save the respective timestamps in these variables
// 2. Fetch data(temp, humid and hi) from backend by calling the getFreqDistro API functions for each
const temp = await AppStore.getFreqDistro("temperature",startDate,endDate);
const humid = await AppStore.getFreqDistro("humidity",startDate,endDate);
const hi = await AppStore.getFreqDistro("heatindex",startDate,endDate);

// 3. create an empty array for each variable (temperature, humidity and heatindex)
// see example below
let temperature = [];
let humidity = [];
let heatindex = [];
// 4. Iterate through the temp variable, which contains temperature data fetched from the backend
// transform the data to {"x": x_value,"y": y_value} format and then push it to the temperature array created previously
// see example below
temp.forEach(row => {
temperature.push({"x": row["_id"],"y": row["count"]})
});
// 5. Iterate through the humid variable, which contains humidity data fetched from the backend
// transform the data to {"x": x_value,"y": y_value} format and then push it to the humidity array created previously
humid.forEach(row => {
humidity.push({"x": row["_id"],"y": row["count"]})
});
// 6. Iterate through the hi variable, which contains heat index data fetched from the backend
humid.forEach(row => {
heatindex.push({"x": row["_id"],"y": row["count"]})
});
// transform the data to {"x": x_value,"y": y_value} format and then push it to the heatindex array created previously
// 7. update series[0] for the histogram/Column chart with temperature data
// see example below
histogramChart.value.series[0].setData(temperature);
// 8. update series[1] for the histogram/Column chart with humidity data
// 9. update series[2] for the histogram/Column chart with heat index data
histogramChart.value.series[1].setData(humidity);
histogramChart.value.series[2].setData(heatindex);
}
}


const initCharts = () => {
  charts.tempHI = Highcharts.chart('container', {
    chart: { type: 'line', zoomType: 'x' },
    title: { text: 'Temperature and Heat Index Analysis', align: 'left' },
    subtitle: { text: 'The heat index, also known as the "apparent temperature," is a measure that combines air temperature and relative humidity to assess how hot it feels to the human body. The relationship between heat index and air temperature is influenced by humidity levels. As humidity increases, the heat index also rises, making the perceived temperature higher than the actual air temperature',align: 'left'  },
    xAxis: { type: 'datetime', title: { text: 'Time' } },
    tooltip: { shared: true, pointFormat: 'Humidity: {point.x} % <br/> Temperature: {point.y} °C'},
    yAxis: { title: { text: 'Celsius (°C)' } },
    series: [{ name: 'Temperature', data: [] }, { name: 'Heat Index', data: [] }]
  });

  Highcharts.chart('container0', {
    chart: { type: 'line', zoomType: 'x' },
    title: { text: 'Humidity Analysis', align: 'left' },
    xAxis: { type: 'datetime', title: { text: 'Time' } },
    yAxis: { title: { text: 'Air Temperature & Heat Index', labels: { format: '{value}°C' } } },
    series: [{ name: 'Humidity', data: [] }],
    tooltip: { shared: true, pointFormat: 'Humidity: {point.x} % <br/> Temperature: {point.y} °C'},
  });

  Highcharts.chart('container1', {
      chart: { type: 'column', zoomType: 'x' },
    title: { text: 'Frequency Distribution Analysis', align: 'left' },
    xAxis: { categories:["Temperature","Humidity","Heat Index"] },
    yAxis: { title: { text: 'Values' } },
    series: [{ name: 'Temperature', data: [] }, { name: 'Humidity', data: [] }, { name: 'Heat Index', data: [] }],
  });

  Highcharts.chart('container2', {
    chart: { type: 'scatter' , zoomType: 'x'},
    title: { text: 'Temperature & Heat Index Correlation Analysis', align: 'left' },
    subtitle:{ text: 'Visualize the relationship between Temperature and Heat Index as well as revealing patterns',align: 'left' },
    xAxis: { title: { text: 'Temperature (°C)', labels: { format: '{value}°C' } } },
    yAxis: { title: { text: 'Heat Index (°C)', labels: { format: '{value}°C' }   } },
    series: [{ name: 'Analysis', data: [] }],
    tooltip: { pointFormat: 'Temperature: {point.x} °C <br/> Heat Index: {point.y} °C' }
  });

  Highcharts.chart('container3', {
    chart: { type: 'scatter', zoomType: 'x' },
    title: { text: 'Humidity & Heat Index Correlation', align: 'left' },
    subtitle:{ text: 'Visualize the relationship between Humidity and Heat Index as well as revealing patterns', align: 'left' },
    xAxis: { title: { text: 'Humidity (%)', labels: { format: '{value}%' } } },
    yAxis: { title: { text: 'Heat Index (°C)', labels: { format: '{value}°C' }   } },
    series: [{ name: 'Analysis', data: [] }]
  });
};

onMounted(() => {
  initCharts();
});
</script>

<style scoped>
.highcharts-figure { margin: 0; }
#container, #container0, #container1, #container2, #container3 {
  width: 100%;
  height: 400px;
  margin-bottom: 20px;
}
</style>