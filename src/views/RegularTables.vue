<template>
    <div>
        <base-header class="pb-6 pb-8 pt-5 pt-md-8 bg-gradient-success">
            <!-- Card stats -->
            <b-row>
                <b-col xl="3" md="6">
                    <stats-card title="Today:" type="gradient-red" class="mb-4">


                        <template slot="footer">
                            <h2>{{ temperature }} <label for="toggle_button">
                                    <span>&#176;{{ isMetric ? "C" : "F" }}</span>

                                    <input type="checkbox" id="toggle_button" v-model="checkedValue">
                                </label></h2>
                            <p>{{ weatherDescription }}</p>

                            <div class="weather-condition">
                                <img v-if="showCloudy" src="../assets/cloudy.png" alt="cloudy">
                                <img v-else-if="showRainy" src="../assets/rain.png" alt="rain">
                                <img v-else-if="showStorm" src="../assets/thunderstorm.png" alt="thunderstorm">
                                <img v-else-if="showClear" src="../assets/clear.png" alt="clear">
                                <img v-else-if="showSnow" src="../assets/snow.png" alt="snow">
                                <img v-else-if="showDrizzle" src="../assets/drizzle.png" alt="drizzle">
                            </div>

                            <div class="input-container">
                                <label for="city">City: </label>
                                <input type="text" name="city" v-model="city">
                            </div>
                        </template>
                    </stats-card>
                </b-col>
                <b-col xl="3" md="6">
                    <table class="weather-condition">
                        <thead>
                            <tr>
                                <th scope="col"></th>
                                <th scope="col">City</th>
                                <th scope="col">Weather today</th>
                                <th scope="col">Temperature today</th>
                                <th scope="col">Weather 24 H</th>
                                <th scope="col">Temperature 24 H</th>
                            </tr>
                        </thead>
                        <tbody>
                            <td scope="row"></td>
                            <td><input readonly type="text" v-model="Forcity" /></td>
                            <td><input readonly type="text" v-model="weatherDescription" /></td>
                            <td><input readonly type="text" v-model="temperature" /></td>
                            <td><input readonly type="text" v-model="ForweatherDescription" /></td>
                            <td><input readonly type="text" v-model="Fortemperature" /></td>
                            <tr v-for="(forecast_data, k) in forecast_datas" :key="k">
                                <button class="ni ni-fat-remove" type='button'
                                    @click="deleteRow(k, forecast_data)"></button>
                                <td><input readonly type="text" v-model="forecast_data.Forcity" /></td>
                                <td><input readonly type="text" v-model="forecast_data.weatherDescription" /></td>
                                <td><input readonly type="text" v-model="forecast_data.temperature" /></td>
                                <td><input readonly type="text" v-model="forecast_data.ForweatherDescription" /></td>
                                <td><input readonly type="text" v-model="forecast_data.Fortemperature" /></td>
                            </tr>
                        </tbody>
                    </table>

                    <button type='button' @click="addNewRow">Add</button>


                    <template slot="footer">
                        <span class="text-success mr-2">54.8%</span>
                        <span class="text-nowrap">Since last month</span>
                    </template>

                </b-col>
            </b-row>
        </base-header>

    </div>
</template>
<script>
import { Dropdown, DropdownItem, DropdownMenu, Table, TableColumn } from 'element-ui';
import projects from './Tables/projects'
import users from './Tables/users'
import LightTable from "./Tables/RegularTables/LightTable";
import DarkTable from "./Tables/RegularTables/DarkTable";
import axios from 'axios'
export default {
    components: {
        LightTable,
        DarkTable,
        [Dropdown.name]: Dropdown,
        [DropdownItem.name]: DropdownItem,
        [DropdownMenu.name]: DropdownMenu,
        [Table.name]: Table,
        [TableColumn.name]: TableColumn
    },
    data() {
        return {
            projects,
            users,
            showCloudy: false,
            showRainy: false,
            showStorm: false,
            showDrizzle: false,
            showClear: false,
            showSnow: false,
            weatherDescription: '',
            ForweatherDescription: '',
            Fortemperature: '',
            city: 'Amsterdam',
            temperature: '',
            Forcity: '',
            forecast_datas: [{}],
            isMetric: true
        };
    },
    computed: {
        checkedValue: {
            get() {
                return this.isMetric
            },
            set(newValue) {
                this.isMetric = newValue
            }
        },
        unit: function () {
            return this.isMetric ? 'metric' : 'imperial'
        }
    },
    watch: {
        city() {
            this.getWeatherAndForecastData()
        },
        unit() {
            this.getWeatherAndForecastData()
        },
    },
    methods: {
        // https://openweathermap.org/weather-conditions
        getWeatherAndForecastData() {
            axios.get('https://api.openweathermap.org/data/2.5/weather?q=' + this.city + '&APPID=94e38cae9ff71a42769f2ff6499340d7&units=' + this.unit).then(
                response => {
                    console.log(this.unit)
                    // console.log(response.data.weather[0].main)
                    let mainDescription = response.data.weather[0].main;
                    let descriptionString = response.data.weather[0].description;

                    this.weatherDescription = descriptionString.charAt(0).toUpperCase() + descriptionString.slice(1);
                    this.temperature = response.data.main.temp.toFixed(1);

                    this.showClear = mainDescription == "Clear"
                    this.showCloudy = mainDescription == "Clouds"
                    this.showRainy = mainDescription == "Rain"
                    this.showDrizzle = mainDescription == "Drizzle" || "Mist"
                    this.showSnow = mainDescription == "Snow"
                    this.showStorm = mainDescription == "Thunderstorm"
                }
            ).catch(error => { console.log(error) })

            axios.get('https://api.openweathermap.org/data/2.5/forecast?q=' + this.city + '&APPID=94e38cae9ff71a42769f2ff6499340d7&units=' + this.unit).then(
                response => {
                    // console.log(response.data)
                    let FordescriptionString = response.data.list[8].weather[0].description;

                    this.ForweatherDescription = FordescriptionString.charAt(0).toUpperCase() + FordescriptionString.slice(1);
                    this.Fortemperature = response.data.list[8].main.temp;
                    this.Forcity = this.city;
                }
            ).catch(error => { console.log(error) }).finally((response) => {
                // always executed
                console.log('HTTP GET Finished!')
            })
        },
        addNewRow() {
            this.forecast_datas.push({
                Forcity: this.Forcity,
                weatherDescription: this.weatherDescription,
                temperature: this.temperature,
                ForweatherDescription: this.ForweatherDescription,
                Fortemperature: this.Fortemperature,
            });
        },
        deleteRow(index, forecast_data) {
            var idx = this.forecast_datas.indexOf(forecast_data);
            console.log(idx, index);
            if (idx > -1) {
                this.forecast_datas.splice(idx, 1);
            }
        },
    },



    mounted() {
        this.getWeatherAndForecastData()
    }
};
</script>

