<template>
  <div class="weather-container">
    <div class="search-box">
      <input
        type="text"
        placeholder="Search..."
        class="search-bar"
        v-model="query"
        v-on:keypress="handleKeyPress"
      />
    </div>
    <div class="weather-details" v-if="typeof weather.main!= 'undefined' ">
      <div class="weather-details__box">
        <img src="../assets/weather-windy.svg" alt="windspeed-icon" style="width:24px;height:24px" />
        <span>{{weather.wind.speed}}m/s</span>
      </div>
      <div class="weather-details__box">
        <img src="../assets/water-percent.svg" alt="humidity-icon" style="width:24px;height:24px" />
        <span>{{weather.main.humidity}}%</span>
      </div>
      <div class="weather-details__box">
        <img src="../assets/cloud.svg" alt="cloud-icon" style="width:24px;height:24px" />
        <span>{{weather.clouds.all}}%</span>
      </div>
    </div>
    <div class="weather-info" v-if="typeof weather.main!= 'undefined' ">
      <div class="weather-info__left-box">
        <div class="flex-column">
          <img :src="weather_icon + weather.weather[0].icon + '.png'" class="weather-icon" />
          <div class="temp">
            {{ getTempDisplay(Math.round(weather.main.temp)) }}
            <span
              class="btn"
              :class="{ 'active': tempUnit === 'Celcius'}"
              @click="setTempUnit('Celcius')"
            >°C</span> |
            <span
              class="btn"
              :class="{ 'active':tempUnit === 'Fahrenheit'}"
              @click="setTempUnit('Fahrenheit')"
            >°F</span>
          </div>
        </div>
      </div>
      <div class="weather-info__right-box">
        <div class="location">{{weather.name}},{{weather.sys.country }}</div>
        <div class="date">{{ todaysDate() }}</div>
        <div class="weather">{{ weather.weather[0].main }}</div>
      </div>
      <div class="weather-box"></div>
    </div>
    <div class="forecast-grid">
      <div v-for="(day) in forecast" :key="day.dt" class="forecast-row">
        <div class="forecast__day">{{ toDayofWeek(day.dt) }}</div>
        <div class="forecast__weather">
          <img :src="weather_icon + day.weather[0].icon + '.png'" class="weather-icon" />
          {{ day.weather[0].description }}
        </div>
        <div class="forecast__temperatures">
          <div>
            {{ getTempDisplay(day.temp.min) }}
            <span v-if="tempUnit === 'Celcius'">°C</span>
            <span v-else>°F</span>
          </div>
          <div>
            {{ getTempDisplay(day.temp.max) }}
            <span v-if="tempUnit === 'Celcius'">°C</span>
            <span v-else>°F</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      api_key: "27bcc252742830381afd6856832da01a",
      url_base: "https://api.openweathermap.org/data/2.5/",
      weather_icon: "http://openweathermap.org/img/wn/",
      query: "",
      weather: {},
      forecast: [],
      tempUnit: "Celcius",
    };
  },
  mounted() {
    this.getLocation();
  },
  methods: {
    handleKeyPress(e) {
      if (e.key === "Enter") {
        this.fetchWeather();
      }
    },
    async fetchWeather() {
      console.log("fetch");
      let current = await axios.get(
        `${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`
      );
      const { lon, lat } = current.data.coord;
      let forecast = await axios.get(
        `${this.url_base}onecall?lat=${lat}&lon=${lon}&appid=${this.api_key}&units=metric`
      );
      this.setResults(current.data, forecast.data.daily);
    },
    setResults(current, forecast) {
      this.weather = current;
      this.forecast = forecast.filter((day, index) => index > 0 && index < 6);
    },
    getTempDisplay(temp) {
      return this.tempUnit === "Fahrenheit"
        ? (temp * 1.8 + 32).toFixed(1)
        : temp.toFixed(1);
    },
    setTempUnit(unit) {
      this.tempUnit = unit;
    },
    toDayofWeek(timestamp) {
      const date = new Date(timestamp * 1000);
      const days = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];

      return days[date.getDay()];
    },
    todaysDate() {
      const months = [
        "Jan",
        "Feb",
        "Mar",
        "Apr",
        "May",
        "Jun",
        "Jul",
        "Aug",
        "Sep",
        "Oct",
        "Nov",
        "Dec",
      ];
      const days = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];

      let d = new Date();
      let month = months[d.getMonth()];
      let day = days[d.getDay()];
      let date = d.getDate();
      let year = d.getFullYear();
      return `${month} ${date} ${day} ${year}`;
    },
    async getLocation() {
      try {
        let response = await axios.get(
          `https://api.ipgeolocation.io/ipgeo?apiKey=3f51feaef697461e8efefe0a440ce808`
        );
        this.query = response.data.city;
        this.fetchWeather();
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Montserrat:wght@300;500;700;900&display=swap");
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Montserrat";
}
.weather-container {
  width: 375px;
  margin: 0 auto;
  border-radius: 25px;
  margin-top: 25px;
  box-shadow: 0px 0px 30px #00000065;
  padding: 10px;
  background-color: black;
}

.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  color: #313131;
  font-size: 20px;
  appearance: none;
  border: none;
  outline: none;
  background: none;
  background-color: rgba(255, 255, 255, 0.5);
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  border-radius: 10px;
  transition: 0.4s;
}
.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
}
.location-box .location {
  color: #fff;
  font-size: 32px;
  font-weight: 500;
  font-style: italic;
  text-align: center;
  margin-top: 30px;
}
.location-box .date {
  color: #fff;
  font-size: 20px;
  font-weight: 300;
  text-align: center;
}
.weather-box {
  text-align: center;
}

.weather-info {
  display: flex;
  flex-wrap: nowrap;
  align-items: center;
  padding: 10px 25px;
  color: #fff;
  font-size: 18px;
  font-weight: 700;
  text-shadow: 2px 2px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 10px 0px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  font-style: italic;
}

.forecast-grid {
  background-color: #4275f5;
  color: white;
  padding: 15px 10px;
  border-radius: 15px;
}

.forecast-row {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
}

.forecast__day {
  width: 15%;
}
.forecast__weather {
  width: 60%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.forecast__temperatures {
  text-align: right;
  width: 25%;
}

.weather-icon {
  width: 70px;
  height: 70px;
}

.weather-info__left-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 50%;
  flex-grow: 1;
}

.weather-info__right-box {
  display: flex;
  flex-direction: column;
  max-width: 50%;
  flex-grow: 1;
}

.weather-details {
  display: flex;
  flex-direction: row;
  color: white;
  justify-content: space-evenly;
  margin-top: 10px;
}

.weather-details__box {
  display: inline-flex;
  align-items: center;
}

.btn:not(.active) {
  color: #70757a;
  cursor: pointer;
}
</style>
