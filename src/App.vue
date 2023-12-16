<script>
import "@/styles.scss";
import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      api_key: "71e050c0438a2e01e18fbd4892746ae3",
      url_base: "https://api.openweathermap.org/data/2.5/weather?",
      url_forecast: "https://api.openweathermap.org/data/2.5/forecast?",
      className: "",
      // greeting: "",
      city: "",
      location: "",
      temp: "",
      tempMax: "",
      tempMin: "",
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
      isError: true,
      query: "",
      quote: "",
      author: "",
      forecast: "",
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
          this.tempMax = response.data.main.temp_max;
          this.tempMin = response.data.main.temp_min;
          this.weekday = this.weekdays[this.date.getDay()];
          this.day = this.date.getDate();
          this.month = this.months[this.date.getMonth()];
          this.loading = false;
          this.noInfo = false;
          this.isError = false;
        })
        .catch((error) => {
          console.log(error);
          this.loading = false;
          this.noInfo = false;
          this.isError = true;
        });
    },
    getForecast(forecastUrl) {
      this.loading = true;
      axios
        .get(forecastUrl)
        .then((response) => {
          this.forecast = response.data.list.filter((e, i) => i % 8 === 8 - 1);
          console.log(this.forecast);
          // const container = document.getElementById("forecast-wrap");
          // forecast.forEach((item) => {
          //   const src = `../dist/img/${item.weather[0].icon}.svg`;
          //   container.innerHTML += `
          // <div>
          //   <div>${item.main.temp.toFixed(1)}°C</div>
          //   <img class="location-img" src=${src} alt="" />
          //   <div>${new Date(item.dt * 1000)
          //     .toLocaleDateString()
          //     .slice(0, 5)}</div>
          //   <div>${this.weekdays[new Date(item.dt * 1000).getDay()].slice(
          //     0,
          //     3
          //   )}</div>
          // </div>`;
          // });
          console.log(this.forecast);
          this.loading = false;
          this.noInfo = false;
          this.isError = false;
        })
        .catch((error) => {
          console.log(error);
          this.loading = false;
          this.noInfo = false;
          this.isError = true;
        });
    },
    getTime() {
      if (this.date.getHours() < 6) {
        // this.greeting = "Good night!";
        this.className = "night";
      } else if (this.date.getHours() < 12) {
        // this.greeting = "Good morning!";
        this.className = "morning";
      } else if (this.date.getHours() < 18) {
        // this.greeting = "Good afternoon!";
        this.className = "day";
      } else if (this.date.getHours() < 24) {
        // this.greeting = "Good evening!";
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
      this.getForecast(
        `${this.url_forecast}lat=${lat}&lon=${lon}&appid=${this.api_key}&units=metric`
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
        this.getForecast(
          `${this.url_forecast}q=${this.query}&appid=${this.api_key}&units=metric`
        );
        this.query = "";
      }
    },
    // getQuote() {
    //   axios
    //     .get("https://api.quotable.io/random")
    //     .then((response) => {
    //       this.quote = response.data.content;
    //       this.author = response.data.author;
    //     })
    //     .catch((error) => {
    //       console.log(error);
    //     });
    // },
  },
  beforeMount() {
    this.geolocation();
    this.getTime();
    // this.getQuote();
  },
};
</script>
<template>
  <div id="bg" v-bind:class="className">
    <main>
      <div v-if="loading" class="custom-loader"></div>
      <div v-else class="forecast-wrap">
        <!-- <div class="greeting">{{ greeting }}</div> -->
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
        <div v-else-if="isError" class="no-info">Something went wrong :(</div>
        <div v-else class="today">
          <div class="day">{{ month }} {{ day }}</div>
          <div class="day">{{ weekday }}</div>
          <div class="location">
            <!-- <img
              class="location-img"
              src="./assets/images/location.svg"
              alt=""
            /> -->
            {{ city }},
            {{ location }}
          </div>
          <div class="temperature-wrap">
            <div class="temp-main">
              <div>{{ Math.round(temp) }}</div>

              <div class="degree">°C</div>
            </div>
            <div class="temp-min-max">
              {{ Math.round(tempMax) }}°C/ {{ Math.round(tempMin) }}°C
            </div>
          </div>
        </div>
        <div class="forecast" id="forecast-wrap">
          <div v-for="item in forecast" class="forecast-item" :key="item">
            <div>
              {{ this.weekdays[new Date(item.dt * 1000).getDay()].slice(0, 3) }}
            </div>
            <img
              class="location-img"
              :src="
                require(`./assets/images/${item.weather[0].icon.slice(
                  0,
                  2
                )}.svg`)
              "
              v-bind:alt="pic"
            />
            <div>{{ item.main.temp.toFixed(1) }}°C</div>
          </div>
        </div>
        <!-- <div class="quotes">
        <div>{{ quote }}</div>
        <div>{{ author }}</div>
      </div> -->
      </div>
    </main>
  </div>
</template>

<style></style>
