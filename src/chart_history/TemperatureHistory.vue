<template>
  <div>
    <input type="date" id="datePicker" v-model="selectedDate" @change="fetchDataForDate" />
    <div>
      <canvas id="TemperatureHistory" ref="chartRef" width="400" height="100"></canvas>
    </div>
  </div>
</template>

<script lang="ts">
import axios from 'axios';
import { ref, onMounted, defineComponent } from 'vue';
import { Chart, registerables } from 'chart.js';
import 'chartjs-adapter-date-fns'; // Import the date-fns adapter
import io from 'socket.io-client';
// @ts-ignore
import annotationPlugin from 'chartjs-plugin-annotation';

Chart.register(...registerables, annotationPlugin);

export default defineComponent({
  name: 'TemperatureHistory',
  setup() {
    const chartRef = ref<HTMLCanvasElement | null>(null);
    const socket = io('http://localhost:3000');
    let temperatureChart: Chart | undefined;
    const selectedDate = ref<string | null>(null);

    const createChart = (data: any) => {
      if (temperatureChart) {
        temperatureChart.destroy();
      }

      if (chartRef.value) {
        const ctx = chartRef.value.getContext('2d');
        if (ctx) {
          temperatureChart = new Chart(ctx, {
            type: 'line',
            data: {
              labels: data.timestamps,
              datasets: [
                {
                  label: 'Sensor 1',
                  backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#009FB7';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#009FB7';
                  },
                  data: data.sensor1_temperature,
                },
                {
                  label: 'Sensor 2',
                  backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#04F06A';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#04F06A';
                  },
                  data: data.sensor2_temperature,
                },
                {
                  label: 'Sensor 3',
                  backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#FED766';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#FED766';
                  },
                  data: data.sensor3_temperature,
                },
                {
                  label: 'Sensor 4',
                  backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#FF0063';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#FF0063';
                  },
                  data: data.sensor4_temperature,
                },
                {
                  label: 'Sensor 5',
                  backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#744FC6';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#744FC6';
                  },
                  data: data.sensor5_temperature,
                },
                {
                  label: 'Sensor 6',
                  backgroundColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#17301C';
                  },
                  borderColor: (context) => {
                    const value = context.raw as number;
                    return value > 65 ? '#DE1A1A' : '#17301C';
                  },
                  data: data.sensor6_temperature,
                },
              ],
            },
            options: {
              responsive: true,
              aspectRatio: 3,
              scales: {
                x: {
                  type: 'time',
                  time: {
                    unit: 'minute',
                    tooltipFormat: 'yyyy-MM-dd HH:mm',
                    displayFormats: {
                      minute: 'HH:mm',
                    },
                  },
                  ticks: {
                    stepSize: 10,
                  },
                  title: {
                    display: true,
                    text: 'Waktu',
                  },
                },
                y: {
                  beginAtZero: true,
                  title: {
                    display: true,
                    text: 'Temperature (°C)',
                  },
                },
              },
              plugins: {
                annotation: {
                  annotations: {
                    threshold: {
                      type: 'line',
                      yMin: 60,
                      yMax: 60,
                      borderColor: '#DE1A1A',
                      borderWidth: 2,
                      borderDash: [6, 6],
                      label: {
                        content: 'Threshold 60°C',
                        display: true,
                        position: 'end',
                      },
                    },
                  },
                },
              },
            },
          });
        }
      }
    };

    const fetchDataForDate = async () => {
      if (selectedDate.value) {
        try {
          const response = await axios.get('http://localhost:3000/api/copy-data');
          const data = response.data;

          // Debugging: Log raw data dari API
          console.log('Raw Data:', data);

          // Filter data berdasarkan tanggal yang dipilih (seluruh hari)
          const startOfDay = new Date(selectedDate.value);
          startOfDay.setHours(0, 0, 0, 0);
          const endOfDay = new Date(startOfDay);
          endOfDay.setHours(23, 59, 59, 999);

          // Debugging: Log start dan end tanggal
          console.log('Start of Day:', startOfDay);
          console.log('End of Day:', endOfDay);

          const filteredData = {
            timestamps: data
              .filter((row: any) => {
                const rowDate = new Date(row.timestamp);
                return rowDate >= startOfDay && rowDate <= endOfDay;
              })
              .map((row: any) => row.timestamp),
            sensor1_temperature: data
              .filter((row: any) => {
                const rowDate = new Date(row.timestamp);
                return rowDate >= startOfDay && rowDate <= endOfDay;
              })
              .map((row: any) => row.sensor1_temperature),
            sensor2_temperature: data
              .filter((row: any) => {
                const rowDate = new Date(row.timestamp);
                return rowDate >= startOfDay && rowDate <= endOfDay;
              })
              .map((row: any) => row.sensor2_temperature),
            sensor3_temperature: data
              .filter((row: any) => {
                const rowDate = new Date(row.timestamp);
                return rowDate >= startOfDay && rowDate <= endOfDay;
              })
              .map((row: any) => row.sensor3_temperature),
            sensor4_temperature: data
              .filter((row: any) => {
                const rowDate = new Date(row.timestamp);
                return rowDate >= startOfDay && rowDate <= endOfDay;
              })
              .map((row: any) => row.sensor4_temperature),
            sensor5_temperature: data
              .filter((row: any) => {
                const rowDate = new Date(row.timestamp);
                return rowDate >= startOfDay && rowDate <= endOfDay;
              })
              .map((row: any) => row.sensor5_temperature),
            sensor6_temperature: data
              .filter((row: any) => {
                const rowDate = new Date(row.timestamp);
                return rowDate >= startOfDay && rowDate <= endOfDay;
              })
              .map((row: any) => row.sensor6_temperature),
          };

          // Debugging: Log data yang difilter
          console.log('Filtered Data:', filteredData);

          createChart(filteredData);
        } catch (error) {
          console.error('Error fetching data for date:', error);
          if (temperatureChart) {
            temperatureChart.destroy();
            temperatureChart = undefined;
          }
        }
      }
    };

    onMounted(() => {
      // Set default date to today
      const today = new Date().toISOString().substr(0, 10);
      selectedDate.value = today;

      // Fetch data for the default date
      fetchDataForDate();
    });

    return {
      chartRef,
      selectedDate,
      fetchDataForDate,
    };
  },
});
</script>

