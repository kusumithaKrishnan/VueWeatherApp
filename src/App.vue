<template>
  <div id="main" >
    <div class="container my-5">
      <h1 class="title text-center">Weather in</h1>
      <form class="search-location" v-on:submit.prevent="getWeather">
        <input
          type="text"
          class="form-control text-muted form-rounded p-4 shadow-sm"
          placeholder="What City?"
          v-model="citySearch"
          autocomplete="off"
        />
        <button v-on:click="currentLocation"> Get Current Location</button>
         
      </form>
      <p class="text-center my-3" v-if="cityFound">No city found</p>
      <div
        class="card rounded my-3 shadow-lg back-card overflow-hidden"
        v-if="visible"
      >
        <!-- weather animation container -->
        <div>
          <div icon="sunny" v-if="clearSky">
            <span class="sun"></span>
          </div>

          <div icon="snowy" v-if="snowy">
            <ul>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
            </ul>
          </div>

          <div icon="stormy" v-if="stormy">
            <span class="cloud"></span>
            <ul>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
            </ul>
          </div>
          <div icon="cloudy" v-if="cloudy">
            <span class="cloud"></span>
            <span class="cloud"></span>
          </div>
          <div icon="nightmoon" v-if="clearNight">
            <span class="moon"></span>
            <span class="meteor"></span>
          </div>
        </div>

        <!-- Top of card starts here -->
        <div class="card-top text-center" style="margin-bottom: 15rem">
          <div class="city-name my-3">
            <p>{{ weather.cityName }}</p>
            <span>...</span>
            <p style="margin-right: 82%">{{ currentTime }}</p>
            <p>Latitude : {{ weather.latitude }}</p>
            <p>Longitude : {{ weather.longitude }}</p>

            <p class="">{{ weather.country }}</p>
          </div>
        </div>
        <!-- top of card ends here -->

        <!--card middle body, card body used cos I want to just update the innerHTML -->
        <div class="card-body">
          <!-- card middle starts here -->
          <div class="card-mid">
            <div class="row">
              <div class="col-12 text-center temp">
                <span>{{ weather.temperature }}&deg;C</span>
                <p class="my-4">{{ weather.description }}</p>
              </div>
            </div>
            <div class="row">
              <div class="col d-flex justify-content-between px-5 mx-5">
                <p>
                  <img src="./assets/down.svg" alt="" />
                  {{ weather.lowTemp }}&deg;C
                </p>
                <p>
                  <img src="./assets/up.svg" alt="" />
                  {{ weather.highTemp }}&deg;C
                </p>
              </div>
            </div>
          </div>
          <!-- card middle ends here -->

          <!-- card bottom starts here -->
          <div class="card-bottom px-5 py-4 row">
            <div class="col text-center">
              <p>{{ weather.feelsLike }}&deg;C</p>
              <span>Feels like</span>
            </div>
            <div class="col text-center">
              <p>{{ weather.humidity }}%</p>
              <span>humidity</span>
            </div>
          </div>

          <!-- card bottom ends here -->
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cityFound: false,
      visible: false,
      stormy: false,
      cloudy: false,
      clearSky: false,
      clearNight: false,
      snowy: false,
      isDay: true,
      citySearch: '',
      currentTime: new Date().toLocaleTimeString(),
      weather: {
        cityName: 'Gwarinpa',
        country: 'NG',
        temperature: 12,
        description: 'Clouds everywhere',
        lowTemp: '19',
        highTemp: '30',
        feelsLike: '20',
        humidity: '55',
        longitude: '11.1',
        latitude: '12.36'
      },
      latitudeData: '',
      longitudeData: '',
      locationDetails: {}
    }
  },
  methods: {
    updateWeather: async function (place) {
      try {
        const weatherData = await this.getStoredCityName(place)
        const {
          name,
          sys: { country },
          coord: { lat, lon },
          main: { temp, temp_min, temp_max, feels_like, humidity }
        } = weatherData
        this.weather = {
          ...this.weather,
          cityName: name,
          country: country,
          temperature: temp,
          lowTemp: temp_min,
          highTemp: temp_max,
          feelsLike: feels_like,
          humidity: humidity,
          longitude: lon,
          latitude: lat
        }
        this.citySearch = ''
        this.weather.description = weatherData.weather[0].description
        const timeOfDay = weatherData.weather[0].icon
        if (timeOfDay.includes('n')) {
          this.isDay = false
        } else {
          this.isDay = true
        }
        return weatherData
      } catch (error) {
        console.log(error)
      }
    },
    getStoredCityName: async function (location) {
      const key = '5299e83c9cf2fef510fb8ddaa3206ac8'
      const baseURL = `http://api.openweathermap.org/data/2.5/weather?q=${location}&appid=${key}&units=metric`
      try {
        const response = await fetch(baseURL)
        const data = await response.json()
        return data
      } catch (error) {
        console.log(error)
      }
    },
    detailedWeather(details) {
      const mainWeather = details.weather[0].main
      //check weather animations
      if (mainWeather.includes('Clouds')) {
        this.stormy = false
        this.cloudy = true
        this.clearSky = false
        this.clearNight = false
        this.snowy = false
      }
      if (mainWeather.includes('Clouds')) {
        this.stormy = false
        this.cloudy = true
        this.clearSky = false
        this.clearNight = false
        this.snowy = false
      }
      if (
        mainWeather.includes('Thunderstorm') ||
        mainWeather.includes('Rain')
      ) {
        this.stormy = true
        this.cloudy = false
        this.clearSky = false
        this.clearNight = false
        this.snowy = false
      }
      if (mainWeather.includes('Clear') && this.isDay) {
        this.stormy = false
        this.cloudy = false
        this.clearSky = true
        this.clearNight = false
        this.snowy = false
      }
      if (mainWeather.includes('Clouds') && !this.isDay) {
        this.stormy = false
        this.cloudy = false
        this.clearSky = false
        this.clearNight = true
        this.snowy = false
      }
      if (mainWeather.includes('Snow')) {
        this.stormy = false
        this.cloudy = false
        this.clearSky = false
        this.clearNight = false
        this.snowy = true
      }
    },
    getWeather: async function () {
      const place = !this.citySearch
        ? localStorage.getItem('cityName')
        : this.citySearch
      try {
        ///check for time of day
        const weatherDetails = await this.updateWeather(place)
        this.detailedWeather(weatherDetails)
        this.visible = true
        this.cityFound = false
      } catch (error) {
        console.log(error)
        this.cityFound = true
        this.visible = false
      }
    },
    getCurrentTime() {
      this.currentTime = new Date().toLocaleTimeString()
    },
    //     showPosition(position) {
    //     let positionData =  position.coords;
    //     return positionData
    //     // this.longitudeData=  position.coords.longitude;
    // },

    getCurrentPosition() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition((pos) => {
          this.locationDetails = pos.coords
          console.log(this.locationDetails)
        })
      }
    },
    getLocation :  async function() {
      const { latitude, longitude } = this.locationDetails
      let apiKey = '0b3ef6279f31441e5f3f566afd388315'
      let baseURLData = `http://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${apiKey}`
      try {
         const response = await fetch(baseURLData)
        const data = await response.json()
        // this.weather = data
        return data

      } catch (error) {
        console.log(error)
      }
    }, 

    currentLocation: function () {
    //  locationDetails(currLocation) {
      // if (currLocation) {
        this.getLocation()
      // }
    // }
    }
  },
  watch: {
    citySearch: function (updatedCity, prevCity) {
      if (!!updatedCity && updatedCity !== prevCity) {
        localStorage.setItem('cityName', updatedCity)
      }
    }
    
  },
  created() {
    this.weather.cityName = localStorage.getItem('cityName')
    setInterval(this.getCurrentTime, 1000)
    this.getWeather()
    this.getCurrentPosition()
  }
}
</script>

<style scoped>
@import './assets/animation.css';
@import './assets/custom.css';

.card-body {
  padding: 6.25rem !important;
}

button {
  float: right;
    margin-top: -39px;
    margin-right: 7px;
    background-color: lightcyan;
    border-radius: 10px;
}
</style>
