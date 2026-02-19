<template>
  <v-container fluid class="bg-surface d-flex justify-center">
    <div id="graph-container" class="pa-5">
      <v-row>
        <v-col cols="12" md="9">
          <figure class="highcharts-figure">
            <div id="container"></div>
          </figure>
        </v-col>

        <v-col cols="12" md="3">
          <v-card class="mb-2" max-width="500px" color="primaryContainer">
            <v-card-subtitle class="text-h6 mt-2">Temperature</v-card-subtitle>
            <v-card-item>
              <span class="text-h3 text-onPrimaryContainer">
                {{ temperature  }}
              </span>
            </v-card-item>
          </v-card>

          <v-card class="mb-2" max-width="500px" color="secondaryContainer">
            <v-card-subtitle class="text-h6 mt-2">Heat Index (Feels like)</v-card-subtitle>
            <v-card-item>
              <span class="text-h3 text-onSecondaryContainer">
                {{ heatindex  }}
              </span>
            </v-card-item>
          </v-card>

          <v-card class="mb-2" max-width="500px" color="tertiaryContainer">
            <v-card-subtitle class="text-h6 mt-2">Humidity</v-card-subtitle>
            <v-card-item>
              <span class="text-h3 text-onTertiaryContainer">
                {{ humidity }}
              </span>
            </v-card-item>
          </v-card>
        </v-col>
      </v-row>

      <v-row justify="start">
        <v-col cols="12" md="9">
          <figure class="highcharts-figure">
            <div id="container1"></div>
          </figure>
        </v-col>
      </v-row>
    </div>
  </v-container>
</template>

<script setup>
import { ref, computed, watch, onMounted, onBeforeUnmount } from "vue";
import Highcharts from 'highcharts';
import more from 'highcharts/highcharts-more';
import Exporting from 'highcharts/modules/exporting';

// Initialize Highcharts modules
more(Highcharts);
Exporting(Highcharts);

// --- REFS & STATE ---
const tempHiChart = ref(null);
const points = ref(10);
const shift = ref(false);



const payload = ref(null); 

// --- COMPUTED ---
const temperature = computed(() => {
  return payload.value ? `${payload.value.temperature.toFixed(2)} °C` : null;
});

const humidity = computed(() => {
  return payload.value ? `${payload.value.humidity.toFixed(2)} %` : null;
});

const heatindex = computed(() => {
  return payload.value ? `${payload.value.heatindex.toFixed(2)} °C` : null;
});

// --- FUNCTIONS ---
const CreateCharts = () => {
  tempHiChart.value = Highcharts.chart('container', {
    chart: { zoomType: 'x' },
    title: { text: 'Air Temperature and Heat Index Analysis', align: 'left' },
    yAxis: {
      title: { text: 'Degrees Celsius', style: { color: '#000000' } },
      labels: { format: '{value} °C' }
    },
    xAxis: {
      type: 'datetime',
      title: { text: 'Time', style: { color: '#000000' } },
    },
    tooltip: { shared: true },
    series: [
      {
        name: 'Temperature',
        type: 'spline',
        data: [],
        color: Highcharts.getOptions().colors[0]
      },
      {
        name: 'Heat Index',
        type: 'spline',
        data: [],
        color: Highcharts.getOptions().colors[3]
      }
    ],
  });

tempHiChart.value = Highcharts.chart('container1', {
    chart: { zoomType: 'x' },
    title: { text: 'Humidity Analysis', align: 'left' },
    yAxis: {
      title: { text: 'Degrees Celsius', style: { color: '#000000' } },
      labels: { format: '{value} °C' }
    },
    xAxis: {
      type: 'datetime',
      title: { text: 'Time', style: { color: '#000000' } },
    },
    tooltip: { shared: true },
    series: [
      {
        name: 'Humidity',
        type: 'spline',
        data: [],
        color: Highcharts.getOptions().colors[0]
      },
      {
        name: 'Heat Index',
        type: 'spline',
        data: [],
        color: Highcharts.getOptions().colors[3]
      }
    ],
  });

};

// --- LIFECYCLE ---
onMounted(() => {
  CreateCharts();
  
  // Note: Ensure Mqtt is imported
  if (typeof Mqtt !== 'undefined') {
    Mqtt.connect();
    setTimeout(() => {
      Mqtt.subscribe("620169500");
    }, 3000);
  }
});

onBeforeUnmount(() => {
  // Cleanup logic if Mqtt needs manual disconnection
});

// --- WATCHERS ---
watch(payload, (data) => {
  if (data && tempHiChart.value) {
    if (points.value > 0) {
      points.value--;
    } else {
      shift.value = true;
    }
    
    const x = data.timestamp * 1000;
    // Series 0: Temperature
    tempHiChart.value.series[0].addPoint({ x, y: parseFloat(data.temperature.toFixed(2)) }, true, shift.value);
    // Series 1: Heat Index (Corrected from humidity to match chart title)
    tempHiChart.value.series[1].addPoint({ x, y: parseFloat(data.heatindex.toFixed(2)) }, true, shift.value);
    // Update Humidity Chart
    tempHiChart.value.series[2].addPoint({ x, y: parseFloat(data.humidity.toFixed(2)) }, true, shift.value);
  }
});
</script>

<style scoped>
#graph-container {
  width: 100%;
  min-height: 600px;
  background-color: #f5f5f5; /* Lightened background for readability */
  border-radius: 8px;
}

.highcharts-figure {
  margin: 0;
  border: 1px solid #ddd;
  background: white;
}

#container, #container1 {
  width: 100%;
  height: 400px;
}
</style>