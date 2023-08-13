<template>
  <div id="bg" v-bind:class="className">
    <main>
      <div v-if="loading" class="custom-loader"></div>
      <div v-else class="forecast-wrap">
        <div class="greeting">{{ greeting }}</div>
        <div v-if="noGeo" class="search-box">
          <input
            type="text"
            class="search-bar"
            placeholder="City..."
            v-model="query"
            @keypress="geoNotAllowed"
          />
        </div>
        <div v-if="noInfo" class="no-info">What's the weather like today?</div>
        <div v-else class="forecast-wrap">
          <div class="day">{{ month }} {{ day }}, {{ year }}</div>
          <div class="day">{{ weekday }}</div>
          <div class="location">
            <img
              class="location-img"
              src="./assets/images/location.svg"
              alt=""
            />
            {{ city }},
            {{ location }}
          </div>
          <div class="temperature">
            {{ Math.round(temp) }}
            <div class="degree">°C</div>
          </div>
          <div class="description">{{ description }}</div>
          <div class="feels-like">
            Feels like {{ Math.round(feelsLikeTemp) }}°C
          </div>
        </div>
      </div>
      <div class="quotes">
        <div>{{ quote }}</div>
        <div>{{ author }}</div>
      </div>
    </main>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      api_key: "71e050c0438a2e01e18fbd4892746ae3",
      url_base: "https://api.openweathermap.org/data/2.5/weather?",
      className: "",
      greeting: "",
      city: "",
      location: "",
      temp: "",
      feelsLikeTemp: "",
      description: "",
      date: new Date(),
      weekdays: [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ],
      weekday: "",
      day: "",
      months: [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ],
      month: "",
      year: "",
      loading: true,
      noGeo: true,
      noInfo: true,
      query: "",
      quote: "",
      author: "",
    };
  },
  methods: {
    getWeather(url) {
      this.loading = true;
      axios
        .get(url)
        .then((response) => {
          this.city = response.data.name;
          this.location = response.data.sys.country;
          this.temp = response.data.main.temp;
          this.feelsLikeTemp = response.data.main.feels_like;
          this.description = response.data.weather[0].description;
          this.weekday = this.weekdays[this.date.getDay()];
          this.day = this.date.getDate();
          this.month = this.months[this.date.getMonth()];
          this.year = this.date.getFullYear();
          this.loading = false;
          this.noInfo = false;
        })
        .catch((error) => {
          console.log(error);
          this.loading = false;
        });
    },
    getTime() {
      if (this.date.getHours() < 6) {
        this.greeting = "Good night!";
        this.className = "night";
      } else if (this.date.getHours() < 12) {
        this.greeting = "Good morning!";
        this.className = "morning";
      } else if (this.date.getHours() < 18) {
        this.greeting = "Good afternoon!";
        this.className = "day";
      } else if (this.date.getHours() < 24) {
        this.greeting = "Good evening!";
        this.className = "evening";
      }
    },
    geolocation() {
      navigator.geolocation.getCurrentPosition(
        this.geoAllowed,
        this.geoNotAllowed
      );
    },
    geoAllowed(position) {
      this.noGeo = false;
      const lat = position.coords.latitude;
      const lon = position.coords.longitude;
      this.getWeather(
        `${this.url_base}lat=${lat}&lon=${lon}&appid=${this.api_key}&units=metric`
      );
      this.noInfo = false;
    },
    geoNotAllowed(e) {
      this.loading = false;
      this.noGeo = true;
      if (e.key == "Enter") {
        this.getWeather(
          `${this.url_base}q=${this.query}&appid=${this.api_key}&units=metric`
        );
        this.query = "";
      }
    },
    getQuote() {
      axios
        .get("https://api.quotable.io/random")
        .then((response) => {
          this.quote = response.data.content;
          this.author = response.data.author;
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
  beforeMount() {
    this.geolocation();
    this.getTime();
    this.getQuote();
  },
};
</script>

<style src="./styles.css"></style>
