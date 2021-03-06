<template>
    <div id="app" class="flex justify-center pt-16">
        <div class="text-white mb-8">
            <div class="places-input text-gray-800">
                <input type="search" id="address" class="form-control rounded-lg" placeholder="In which city do you live?" />

                <p>Selected: <strong id="address-value">none</strong></p>
            </div>

            <div class="weather-container font-sans w-128 max-w-lg rounded-lg overflow-hidden bg-gray-900 shadow-lg mt-4">
                <div 
                    v-for="item in current.currentWeather"
                    :key="item"
                    class="current-weather flex items-center justify-between px-6 py-8"
                >
                    <div class="flex item-center">
                        <div>
                            <div class="text-6xl font-semibold">{{ currentTemp.actual }}°C</div>
                            <div>Feels like {{ currentTemp.feels_like }}°C</div>
                        </div>

                        <div class="mx-5 mt-4">
                            <div class="font-semibold capitalize">{{ item.description }}</div>
                            <div>{{ location.name }}</div>
                        </div>
                    </div>

                    <img v-bind:src="'http://openweathermap.org/img/wn/' + item.icon + '@2x.png'" alt="">
                </div>

                <div class="future-weather text-sm bg-gray-800 px-6 py-8 overflow-hidden">
                    <div 
                        v-for="(day, index) in daily"
                        :key="index"
                    >
                        <div
                            class="flex items-center"
                            v-if="index < 5" 
                            :class="{ 'mt-8' : index > 0 }" 
                        >
                            <div class="w-1/6 text-lg text-gray-200">{{ toDayOfWeek(day.dt) }}</div>
                            <div 
                                v-for="item in day.weather"
                                :key="item"
                                class="w-4/6 px-4 flex items-center"
                            >
                                <img v-bind:src="'http://openweathermap.org/img/wn/' + item.icon + '.png'" alt="">

                                <div class="ml-3 capitalize">{{ item.description }}</div>
                            </div>
                            <div class="w-1/6 text-right">
                                <div>{{ Math.round(day.temp.max) }}°C</div>
                                <div>{{ Math.round(day.temp.min) }}°C</div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        mounted() {
            this.fetchData()

            var placesAutocomplete = places({
                appId: 'plTTN6NMFY49',
                apiKey: 'e11904ffe1c397f1a8e5015d265ed8cd',
                container: document.querySelector('#address'),
            }).configure({
                type: 'city',
                aroundLatLngViaIP: false,
            });

            var $address = document.querySelector('#address-value')

            placesAutocomplete.on('change', (e) => {
                console.log(e.suggestion);
                $address.textContent = e.suggestion.value

                this.location.name = `${e.suggestion.name}, ${e.suggestion.country}`
                this.location.lat = e.suggestion.latlng.lat
                this.location.lon = e.suggestion.latlng.lng
            });

            placesAutocomplete.on('clear', function() {
                $address,textContent = 'none';
            });
        },

        watch: {
            location: {
                handler(newValue, oldValue) {
                    this.fetchData()
                },
                deep: true
            }
        },

        data() {
            return {
                currentTemp: {
                    actual: '',
                    feels_like: '',
                },

                current: {
                    currentWeather: [],
                },
                

                daily: [],

                location: {
                    name: 'Bath, UK',
                    lat: 51.3779,
                    lon: -2.3591,
                }
            }
        },

        methods: {
            fetchData() {
                fetch(`/weather?lat=${this.location.lat}&lon=${this.location.lon}`)
                .then(response => response.json())
                .then(data => {
                    this.currentTemp.actual = Math.round(data.current.temp)
                    this.currentTemp.feels_like = Math.round(data.current.feels_like)

                    this.current.currentWeather = data.current.weather

                    this.daily = data.daily                    
                })
            },
            toDayOfWeek(timestamp) {
                const newDate = new Date(timestamp*1000)
                const days = ['SUN', 'MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT']

                return days[newDate.getDay()]
            }
        },
    }
</script>
