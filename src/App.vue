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
      city: "",
      location: "",
      temp: "",
      description: "",
      details: "",
      wind: "",
      humidity: "",
      pressure: "",
      visibility: "",
      temp_max: "",
      temp_min: "",
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
        "Jan",
        "Feb",
        "Mar",
        "Apr",
        "May",
        "Jun",
        "Jul",
        "Aug",
        "Sept",
        "Oct",
        "Nov",
        "Dec",
      ],
      month: "",
      year: "",
      loading: true,
      noGeo: true,
      noInfo: true,
      isError: true,
      query: "",
      forecast: "",
      icon: "",
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
          this.description = response.data.weather[0].main;
          this.details = response.data.weather[0].description;
          this.wind = response.data.wind.speed.toFixed(1);
          this.humidity = response.data.main.humidity;
          this.pressure = response.data.main.grnd_level;
          this.visibility = response.data.visibility / 1000;
          this.temp_max = Math.round(response.data.main.temp_max);
          this.temp_min = Math.round(response.data.main.temp_min);
          this.weekday = this.weekdays[this.date.getDay()];
          this.day = this.date.getDate();
          this.month = this.months[this.date.getMonth()];
          this.icon = response.data.weather[0].icon.slice(0, 2);
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
        this.className = "night";
      } else if (this.date.getHours() < 12) {
        this.className = "morning";
      } else if (this.date.getHours() < 18) {
        this.className = "day";
      } else if (this.date.getHours() < 24) {
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
        <details v-else class="today">
          <summary>
            <div class="top-info">
              <div>{{ month }} {{ day }}</div>
              <div>{{ weekday }}</div>
              <div>
                {{ city }},
                {{ location }}
              </div>
            </div>
            <img
              class="today-img"
              :src="require(`./assets/images/${icon}.svg`)"
              v-bind:alt="pic"
            />
            <div class="temperature-wrap">
              <div class="temp-main">
                <div>{{ Math.round(temp) }}</div>
                <div class="degree">°C</div>
              </div>
            </div>
            <div class="description">
              {{ description }}
            </div>
            <div class="details-modal-overlay"></div>
          </summary>
          <div class="details-modal">
            <div class="details-modal-title">{{ details }}</div>
            <div class="details-modal-content">
              <div class="details-modal-content-info">
                <img
                  class="details-modal-img"
                  src="./assets/images/wind.svg"
                  alt=""
                />
                <p>{{ wind }} m/s</p>
              </div>
              <div class="details-modal-content-info">
                <img
                  class="details-modal-img"
                  src="./assets/images/humidity.svg"
                  alt=""
                />
                <p>{{ humidity }}%</p>
              </div>
              <div class="details-modal-content-info">
                <img
                  class="details-modal-img"
                  src="./assets/images/pressure.svg"
                  alt=""
                />
                <p>{{ pressure }} hPa</p>
              </div>
              <div class="details-modal-content-info">
                <img
                  class="details-modal-img"
                  src="./assets/images/visibility.svg"
                  alt=""
                />
                <p>{{ visibility }} km</p>
              </div>
              <div class="details-modal-content-info">
                <img
                  class="details-modal-img"
                  src="./assets/images/temp-max.svg"
                  alt=""
                />
                <p>{{ temp_max }}°C</p>
              </div>
              <div class="details-modal-content-info">
                <img
                  class="details-modal-img"
                  src="./assets/images/temp-min.svg"
                  alt=""
                />
                <p>{{ temp_min }}°C</p>
              </div>
            </div>
          </div>
        </details>
        <div class="forecast" id="forecast-wrap">
          <details v-for="item in forecast" class="forecast-item" :key="item">
            <summary>
              <div>
                {{
                  this.weekdays[new Date(item.dt * 1000).getDay()].slice(0, 3)
                }}
              </div>
              <img
                class="forecast-img"
                :src="
                  require(`./assets/images/${item.weather[0].icon.slice(
                    0,
                    2
                  )}.svg`)
                "
                v-bind:alt="pic"
              />
              <div>{{ item.main.temp.toFixed(0) }}°C</div>
              <div class="details-modal-overlay"></div>
            </summary>
            <div class="details-modal">
              <div class="details-modal-title">
                {{
                  new Date(item.dt * 1000).toDateString().slice(3, 11).trim()
                }},
                {{ new Date(item.dt * 1000).toDateString().slice(0, 3) }}
              </div>
              <div class="details-modal-content">
                {{ item.weather[0].description }}
                <div class="details-modal-content-info">
                  <img
                    class="details-modal-img"
                    src="./assets/images/wind.svg"
                    alt=""
                  />
                  <p>{{ item.wind.speed.toFixed(1) }} m/s</p>
                </div>
                <div class="details-modal-content-info">
                  <img
                    class="details-modal-img"
                    src="./assets/images/humidity.svg"
                    alt=""
                  />
                  <p>{{ item.main.humidity }}%</p>
                </div>
                <div class="details-modal-content-info">
                  <img
                    class="details-modal-img"
                    src="./assets/images/pressure.svg"
                    alt=""
                  />
                  <p>{{ item.main.grnd_level }} hPa</p>
                </div>
                <div class="details-modal-content-info">
                  <img
                    class="details-modal-img"
                    src="./assets/images/visibility.svg"
                    alt=""
                  />
                  <p>{{ item.visibility / 1000 }} km</p>
                </div>
                <div class="details-modal-content-info">
                  <img
                    class="details-modal-img"
                    src="./assets/images/temp-max.svg"
                    alt=""
                  />
                  <p>{{ Math.round(item.main.temp_max) }}°C</p>
                </div>
                <div class="details-modal-content-info">
                  <img
                    class="details-modal-img"
                    src="./assets/images/temp-min.svg"
                    alt=""
                  />
                  <p>{{ Math.round(item.main.temp_min) }}°C</p>
                </div>
              </div>
            </div>
          </details>
        </div>
      </div>
    </main>
  </div>
</template>
