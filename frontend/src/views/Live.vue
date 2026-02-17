<template>
    <v-container fluid class="bg-surface d-flex justify-center" >
        <div id="graph-container">
            <v-row>
                <v-col cols="9">
                    <figure class="highcharts-figure">
                        <div id="container">

                        </div>
                    </figure>
                </v-col>
                <v-col cols="3">
                    <v-card margin-bottom=5px; Max-width= 500px; color= primaryContainer;>
                        <v-card-subtitle class="text-h6">Temperature</v-card-subtitle>
                        
                            <div id="temp-container">
                                <v-card-item>
                                    <span class="text-h3 text-onPrimaryContainer">
                                       {{ temperature }}
                                    </span>
                                </v-card-item>
                                
                            </div>
                        
                        
                    </v-card>
                     <v-card margin-bottom=5px; Max-width= 500px; color= secondaryContainer;>
                        <v-card-subtitle class="text-h6">Heat Index (Feels like) </v-card-subtitle>
                        
                            <div id="heatindex-container">
                                  <v-card-item>
                                    <span class="text-h3 text-onSecondaryContainer">
                                       
                                        {{ heatindex }}
                                       

                                    </span>
                                </v-card-item>
                            </div>
                        
                       

                    </v-card>
                     <v-card margin-bottom=5px; Max-width= 500px; color= tertiaryContainer;>
                        <v-card-subtitle class="text-h6">Humidity</v-card-subtitle>
                        
                            <div id="humidity-container">
                                  <v-card-item>
                                    <span class="text-h3 text-onTertiaryContainer">
                                     
                                        {{ humidity }}

                                    </span>
                                </v-card-item>
                            </div>

                    </v-card>
                    
                </v-col>
            </v-row>
            <v-row justify="start">
                <v-col cols="9">
                    <figure class="highcharts-figure">
                        <div id="container1">
                            
                        </div>
                    </figure>
                </v-col>
            </v-row>
        </div>
         
       
    </v-container>
</template>

<script setup>
/** JAVASCRIPT HERE */

// IMPORTS
import Highcharts from 'highcharts';
import more from 'highcharts/highcharts-more';
import Exporting from 'highcharts/modules/exporting';
Exporting(Highcharts);
more(Highcharts);
import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
 
 
// VARIABLES
const router      = useRouter();
const route       = useRoute();  
const tempHiChart = ref(null); // Chart object
const points = ref(10); // Specify the quantity of points to be shown on the live graph simultaneously.
const shift = ref(false); // Delete a point from the left side and append a new point to the right side of the graph.
const temperature = computed(()=>{
if(!!payload.value){
return `${payload.value.temperature.toFixed(2)} °C`;
}});
const humidity = computed(()=>{
if(!!payload.value){
return `${payload.value.humidity.toFixed(2)} %`;
}});
const heatindex = computed(()=>{
if(!!payload.value){
return `${payload.value.heatindex.toFixed(2)} °C`;
}
});

// FUNCTIONS
onMounted(()=>{
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    CreateCharts();
Mqtt.connect(); // Connect to Broker located on the backend
setTimeout( ()=>{
// Subscribe to each topic
Mqtt.subscribe("620169500");
Mqtt.subscribe("topic2");
},3000);
});



onBeforeUnmount(()=>{
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
});

const CreateCharts = async () => {
// TEMPERATURE CHART
tempHiChart.value = Highcharts.chart('container', {
chart: { zoomType: 'x' },
title: { text: 'Air Temperature and Heat Index Analysis', align: 'left' },
yAxis: {
title: { text: 'Air Temperature & Heat Index' , style:{color:'#000000'}},
labels: { format: '{value} °C' }
},
xAxis: {
type: 'datetime',
title: { text: 'Time', style:{color:'#000000'} },
},
tooltip: { shared:true, },
series: [
{
name: 'Temperature',
type: 'spline',
data: [],
turboThreshold: 0,
color: Highcharts.getOptions().colors[0]
},
{
name: 'Humidity',
type: 'spline',
data: [],
turboThreshold: 0,
color: Highcharts.getOptions().colors[1]
} ],
});
};

// WATCHERS
watch(payload,(data)=> {
if(points.value > 0){ points.value -- ; }
else{ shift.value = true; }
tempHiChart.value.series[0].addPoint({y:parseFloat(data.temperature.toFixed(2)) ,x: data.timestamp * 1000 }, true, shift.value);
tempHiChart.value.series[1].addPoint({y:parseFloat(data.humidity.toFixed(2)) ,x: data.timestamp * 1000 }, true, shift.value);

})
</script>


<style scoped>
/** CSS STYLE HERE */
#graph-container{
    width: 100%;
    height: 400px;
    background-color: #102066;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 8px;
}


Figure {
border: 2px solid black;
}

</style>
  