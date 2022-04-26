<template>
    <ion-page>
        <ion-header :translucent="true">
            <ion-toolbar class="new-background-color">
                <ion-title size="large">Ma météo</ion-title>
            </ion-toolbar>
        </ion-header>

        <ion-content :fullscreen="true">
            <ion-header collapse="condense">
                <ion-toolbar>
                    <ion-title size="large">Blank</ion-title>
                </ion-toolbar>
            </ion-header>

            <div id="date">
                <p class="current-date"> {{currentDate}} </p>
            </div>

            <div>
                <ion-item>
                    <ion-label>Ville</ion-label>
                    <ion-select placeholder="Choicer" v-model="placeIndex" @ionChange="placeSelection">
                        <ion-select-option :value="index" v-for="(city, index) in cities" :key="index">{{city.fr}}
                        </ion-select-option>
                    </ion-select>
                </ion-item>
            </div>
            <div v-if="place!==''" class="init-weather ion-text-center">
                <div class="ion-text-center">
                    <h1>{{place}}</h1>
                </div>

                <div>
                    <h2>{{temperature}} &deg;C</h2>
                </div>
                <div v-if="icon!==''">
                    <ion-img :src="icon" class="image"></ion-img>
                </div>
                <div>
                    <h3>{{description}}</h3>
                </div>
                <div style="color: #939aa7; padding-top: 1rem">
                    <h3> Latitude: {{lat}}</h3>
                    <h3> Longitude: {{lon}}</h3>
                </div>
            </div>
            <div v-else class="ion-text-center init-weather">
                <h2 class="init-weather-header1"> Veuillez sélectionner une</h2>
                <h1 class="init-weather-header2">ville</h1>
            </div>

        </ion-content>
        <ion-footer collapse="fade">
            <ion-toolbar class="new-background-color">
                <ion-title>&#169; Philippe Mudaheranwa</ion-title>
            </ion-toolbar>
        </ion-footer>
    </ion-page>
</template>

<script lang="ts">
    import {IonContent, IonHeader, IonPage, IonSelect, IonSelectOption, IonTitle, IonToolbar} from '@ionic/vue';
    import {defineComponent} from 'vue';
    import moment from 'moment';
    import axios from 'axios';

    moment.locale('fr');

    export default defineComponent({
        name: 'HomePage',
        components: {
            IonContent,
            IonHeader,
            IonPage,
            IonTitle,
            IonToolbar,
            IonSelect,
            IonSelectOption
        },
        data() {
            return {
                currentDate: moment().format("dddd, DD MMMM YYYY"),
                cities: [
                    {en: "Montreal", fr: "Montréal"},
                    {en: "Laval", fr: "Laval"},
                    {en: "Quebec", fr: "Québec"},
                    {en: "CurrentPlace", fr: "Position actuelle"}
                ],
                placeIndex: -1,
                place: "",
                temperature: 0,
                icon: "",
                description: "",
                lat: 0,
                lon: 0
            }
        },
        mounted(): void {
            this.placeIndex = 0;
            this.placeSelection();
        },
        methods: {
            placeSelection() {
                this.place = this.cities[this.placeIndex]["fr"];
                this.getGivenLocationWeather(this.cities[this.placeIndex]["en"]);
            },
            getGivenLocationWeather(city: string) {
                if (city !== "CurrentPlace") {
                    this.weatherEndpoint(city);
                } else {
                    this.getCurrentLocationWeather();
                }

            },
            getCurrentLocationWeather() {
                fetch('https://api.ipify.org?format=json')
                    .then(x => x.json())
                    .then(({ip}) => {

                        axios.get(`https://ipapi.co/${ip}/json/`)
                            .then(response => {
                                this.place = response.data.city;
                                this.weatherEndpoint(response.data.city);
                            })
                            .catch(e => {
                                console.log("Error", e)
                            })
                    });
            },
            weatherEndpoint(city: string) {
                const lang = "fr";
                axios.get(`https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=1663aa359569ab7c9c628c9df1aeb488&lang=${lang}&units=metric`)
                    .then(response => {
                        if (JSON.stringify(response.data) !== '{}') {
                            this.lat = response.data.coord.lat;
                            this.lon = Math.abs(response.data.coord.lon);
                            this.temperature = response.data.main.temp;
                            this.icon = `http://openweathermap.org/img/wn/${response.data.weather[0]["icon"]}@2x.png`;
                            this.description = response.data.weather[0]["description"];
                        }
                    }).catch(e => {
                        console.log("Error", e);
                    });
            }
        }
    });
</script>

<style scoped>

    #date {
        text-align: center;
        font-weight: bold;
        font-size: large;
    }

    #container strong {
        font-size: 20px;
        line-height: 26px;
    }

    #container p {
        font-size: 16px;
        line-height: 22px;

        color: #8c8c8c;

        margin: 0;
    }

    #container a {
        text-decoration: none;
    }

    .current-date {
        font-size: 1.3rem;
        color: #626664;
    }

    .image {
        height: 9rem !important;
    }

    .init-weather {
        padding-top: 1rem;
    }

    .init-weather-header1 {
        color: #7e7e7e;
        font-size: 1.8rem;
        font-family: serif;
    }

    .init-weather-header2 {
        color: #7e7e7e;
        font-size: 2.5rem;
        font-family: serif;
    }

    .new-background-color {
        --background: #3ba2ff;
        color: white;
    }
</style>
