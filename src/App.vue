<template>
  <div id="app" :class="typeof weather.main != 'undefined' && weather.main.temp > 16 ? 'warm' : ''">
    <main>
      <div class="search-box">
        <input 
          type="text" 
          class="search-bar" 
          placeholder="Search..."
          v-model="query"
          @keypress.enter="fetchWeather">
      </div>
      <div class="weather-wrap" v-if="typeof weather.main != 'undefined'">
        <div class="location-box">
          <div class="location">{{ weather.name }}, {{ weather.sys.country }}</div>
          <div class="date"> {{ dateBuilder() }} </div>
        </div>
        <div class="weather-box">
          <div class="temp">{{ Math.round(weather.main.temp) }}°C</div>
          <div class="weather">
            <span>{{ weather.weather[0].main }} </span>
            <span>
              <img :src="getIconUrl()" 
                   :alt="weather.weather[0].description">
            </span>
          </div>
        </div>
        <div class="details">Details</div>
        <div class="weather-details">
          <div class="feel-like">Feel like<br>{{ Math.round(weather.main.feels_like) }}°C</div>
          <div class="wind-speed">Wind<br> {{ weather.wind.speed }} m/s {{ getCardinalDirection() }}</div>
          <div class="humidity">Humidity<br> {{ weather.main.humidity }} %</div>
          <div class="pressure">Pressure<br> {{ weather.main.pressure }} hPa</div>
        </div>
        <button @click="fetchWeatherForecast" class="forecast-btn">Get forecast</button>
        <div class="weather-forecast">
          <ul v-for="i in weatherForecast" :key="i.dt">
            <li>{{ i.dt | toDate }} - {{ Math.round(i.temp.day) }}°C/{{ Math.round(i.temp.night) }}°C - {{i.weather[0].main}} - {{Math.round(i.wind_speed)}} m/s </li>
          </ul>
        </div>

      </div>
    </main>
  </div>
</template>

<script>


export default {
  name: 'App',
  data(){
    return {
      api_key: '8e83ae42a8923290dcd52d4f7c42500d',
      url_base: 'https://api.openweathermap.org/data/2.5',
      url_forecast: 'https://api.openweathermap.org/data/2.5/onecall?',
      url_icon: 'http://openweathermap.org/img/wn/',
      query: '',
      weather: {},
      weatherForecast: {}
    }
    
  },
  methods: {
    fetchWeather() {
      fetch(`${this.url_base}/weather?q=${this.query}&units=metric&appid=${this.api_key}`)
        .then(res => {
          return res.json()
        })
        .then(this.setResault);
    },
    fetchWeatherForecast() {
      fetch(`${this.url_forecast}lat=${this.weather.coord.lat}&lon=${this.weather.coord.lon}&exclude=hourly&appid=${this.api_key}&units=metric`)
        .then(res => {
          return res.json()
        })
        .then(this.setForecastResault);
    },
    setResault(results){
      this.weather = results
    },
    setForecastResault(results) {
      this.weatherForecast = results.daily
    },
    dateBuilder() {
      let date = new Date();
      let months= ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];
      let days = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];

      let d = date.getDate();
      let day = days[date.getDay()];
      let month = months[date.getMonth()];
      let year = date.getFullYear();

      return `${day} ${d} ${month} ${year}`;
    },
    getCardinalDirection() {
      const directions = ['↑ N', '↗ NE', '→ E', '↘ SE', '↓ S', '↙ SW', '← W', '↖ NW'];
      return directions[Math.round(this.weather.wind.deg / 45) % 8];
    },
    getIconUrl(){
      return `${this.url_icon}${this.weather.weather[0].icon}@2x.png`
    }
  },
  filters: {
    toDate(timestamp){
      let date = new Date(timestamp * 1000)
      console.log(date);
      let days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
      let d = days[date.getDay()];
      let day = date.getDate();
      let month = date.getMonth();
      return `${d} ${day}.${month + 1}`;
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: 'montserrat', sans-serif;
}
#app {
  background-image: url('./assets/cold-bg.jpg');
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}
#app.warm{
  background-image: url('./assets/warm-bg.jpg');
}
main{
  min-height: 100vh;
  padding: 25px;
  background-image: linear-gradient(to bottom, rgba(0,0,0,.25), rgba(0,0,0,.75)); 
}
.search-box{
  width: 100%;
  margin-bottom: 30px;
}
.search-box .search-bar{
  display: block;
  width: 100%;
  padding: 15px;
  color: #313131;
  font-size: 20px;
  border: none;
  outline: none;
  appearance: none;
  background: none;
  box-shadow: 0 0 8px rgba(0,0,0,.25);
  background-color: rgba(255, 255, 255, .5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}
.search-box .search-bar:focus{
   box-shadow: 0 0 16px rgba(0,0,0,.25);
   background-color: rgba(255, 255, 255, .75);
   border-radius: 16px 0px 16px 0px;
}

.location-box .location{
  color: #fff;
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0,0,0,.25);
}
.location-box .date{
  color: #fff;
  font-size: 20px;
  font-weight: 300;
  text-align: center;
  font-style: italic;
}
.weather-box{
  text-align: center;
}
.weather-box .temp{
  display: inline-block;
  padding: 10px 25px;
  color: #fff;
  font-size: 100px;
  font-weight: 900;

  text-shadow:3px 6px rgba(0,0,0,.25);
  background-color: rgba(255, 255, 255, .25);
  border-radius: 16px;
  margin: 30px 0;

  box-shadow: 3px 6px rgba(0, 0, 0, .25);
}

.weather-box .weather{
  color: #fff;
  font-size: 48px;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0,0,0,.25);
}
.details{
  color: #fff;
  text-decoration: underline;
  font-size: 20px;
}
.weather-details{
  color: #fff;
  display: -webkit-flex;
  display: -moz-flex;
  display: -ms-flex;
  display: -o-flex;
  display: flex;
  justify-content: space-between;
  margin-top: 10px;
}
.weather-details div{
  padding: 5px 5px;
  background-color: rgba(255, 255, 255, .25);
  box-shadow: 1px 3px rgba(0, 0, 0, .25);
  text-align: center;
}
.forecast-btn{
  margin: 10px auto;
  padding: 10px 20px;
  border-radius: 5px;
  font-size: 18px;
  color: #fff;
  cursor: pointer;
  outline: none;
  border: none;
  background-color: rgba(255, 255, 255, .25);
  transition: .5s;
}
.forecast-btn:hover{
  background-color: rgba(255, 255, 255, .55);
  border-radius: 10px;
}
.weather-forecast{
  color: #fff;
}
.weather-forecast ul{
  list-style: none;
}
.weather-forecast ul li{
  padding: 10px 10px;
  border: 1px solid rgba(255, 255, 255, .25);
  border-radius: 5px;
  margin-bottom: 5px;

}
</style>
