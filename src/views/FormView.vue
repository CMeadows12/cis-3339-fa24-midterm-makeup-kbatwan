// asked ChatGPT to help me make sure I imported all the necessary packages 
<template>
    <div>
        <form @submit.prevent="fetchRemoteData">
            <div class="form-group">
                <h2>Weather Data</h2>
                <p>Please fill in the form and use the submit button to load data.</p>
                <label for="">Location</label>
                <br>
                <input class="form-control rounded" v-model="formInput.location" type="text" required />
            </div>

            <div class="form-group">
                <label for="">Start Date</label>
                <br>
                <input class="form-control rounded" type="date" v-model="formInput.startDt" required />
            </div>

            <div class="form-group">
                <label for="">End Date</label>
                <br>
                <input class="form-control rounded" type="date" v-model="formInput.endDt" required />
            </div>
            <br>
            <div>
                <button class="btn btn-danger" type="submit">Submit</button>
            </div>
        </form>

        <br /><br />

        <div v-if="dataLoaded">
            <Bar :data="chartData" :options="chartOptions" />
        </div>

        <div v-if="dataLoaded">
            <hr>
            <h5>Weather Data in a Table</h5>
            <table class="table">
                <thead>
                    <tr>
                        <th v-for="date in keys" :key="date">{{ date }}</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td v-for="temp in values" :key="temp">{{ temp }}</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<style>
table, th, td {
  border: 1px solid black;
  border-collapse: collapse;
}
</style>

<script>
import axios from "axios";
import {
    Chart as ChartJS,
    Title,
    Tooltip,
    Legend,
    BarElement,
    CategoryScale,
    LinearScale
} from 'chart.js'
import { Bar } from 'vue-chartjs'
import { ref, reactive } from 'vue'

ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend)

export default {
    components: {
        Bar
    },

    setup() {
        // Reactive form input for capturing user input
        const formInput = reactive({
            location: "",
            startDt: "",
            endDt: ""
        });

        // Ref for controlling when data is displayed
        const dataLoaded = ref(false);

        // Chart options configuration
        const chartOptions = {
            responsive: true,
            maintainAspectRatio: true,
            scales: {
                x: {
                    title: {
                        display: true,
                        text: "Time"
                    }
                },
                y: {
                    title: {
                        display: true,
                        text: "Temperature (F)"
                    }
                }
            }
        };

        // Reactive data for chart and table
        const keys = ref([]);
        const values = ref([]);
        const chartData = ref({
            labels: [],
            datasets: [{
                label: "Average Temperature",
                backgroundColor: "#f87979",
                data: []
            }]
        });

        // Function to fetch weather data
        const fetchRemoteData = async () => {
            const options = {
                method: 'GET',
                url: 'https://weatherapi-com.p.rapidapi.com/history.json',
                params: {
                    q: formInput.location,
                    dt: formInput.startDt,
                    end_dt: formInput.endDt,
                    lang: 'en'
                },
                headers: {
                    'X-RapidAPI-Key': '6c6f790650msh5ce4da2fced77a7p1b1776jsn247d9f531b74',
                    'X-RapidAPI-Host': 'weatherapi-com.p.rapidapi.com'
                }
            };

            try {
                const response = await axios.request(options);
                const weatherData = response.data.forecast.forecastday;
                keys.value = weatherData.map(item => item.date);
                values.value = weatherData.map(item => item.day.avgtemp_f);

                chartData.value = {
                    labels: keys.value,
                    datasets: [{
                        label: "Average Temperature",
                        backgroundColor: "#98FF98", // change color to mint 
                        data: values.value
                    }]
                };

                dataLoaded.value = true; // Set dataLoaded to true to display chart and table
            } catch (error) {
                console.error(error);
            }
        };

        return {
            formInput,
            chartOptions,
            chartData,
            dataLoaded,
            keys,
            values,
            fetchRemoteData
        };
    }
}
</script>
