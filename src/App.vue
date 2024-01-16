<template>
  <div id="app" :class="typeof weather.main != 'undefined'" :style="{ backgroundImage: 'url(' + imageUrl + ')' }">
    <main>
      <div class="search-box">
        <input type="text" class="search-bar" placeholder="Search..." v-model="query" @keypress="handleKeyPress" />
      </div>
      <div class="weather-wrap" v-if="typeof weather.main != 'undefined'">
        <div class="location-box">
          <div class="location">{{ weather.name }}, {{ weather.sys.country }}</div>
          <div class="date">{{ dateBuilder() }}</div>
        </div>

        <div class="weather-box">
          <div class="temp">{{ weather.main.temp }}°C</div>
          <div class="weather">{{ weather.weather[0].main }} </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import env from './env.js'

const api_key = env.api_key;
const url_base = 'https://api.openweathermap.org/data/2.5/weather';
let query = ref('New York');
let weather = ref({});
const fetchWeather = () => {
  fetch(`${url_base}?q=${query.value}&units=metric&APPID=${api_key}`)
    .then(res => res.json())
    .then((results) => {
      setResults(results);
    });
  query.value = '';
}

const setResults = (results) => {
  weather.value = results;
}

//Access Location 
const location = ref({});
const error = ref(null);

const getCurrentLocation = () => {
  return new Promise((resolve, reject) => {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          location.value = {
            latitude: position.coords.latitude,
            longitude: position.coords.longitude,
          };
          resolve(location.value);
        },
        (err) => {
          error.value = `Error getting location: ${err.message}`;
          reject(error.value);
        }
      );
    } else {
      error.value = 'Geolocation is not supported by your browser.';
      reject(error.value);
    }
  });
};


//convert coordinates to city
const getCityName = async (latitude, longitude) => {
  const apiKey = env.apikey;
  const response = await fetch(
    `https://api.opencagedata.com/geocode/v1/json?q=${latitude}+${longitude}&key=${apiKey}`
  );
  const data = await response.json();

  if (data.results.length > 0) {
    return data.results[0].components.city;
  } else {
    throw new Error('City not found');
  }
};

const handleKeyPress = (e) => {
  if (e.key === "Enter") {
    showTitle();
    fetchWeather();
    fetchImage();
  }
};

const dateBuilder = () => {
  let d = new Date();
  let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
  let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];

  let day = days[d.getDay()];
  let date = d.getDate();
  let month = months[d.getMonth()];
  let year = d.getFullYear();

  return `${day} ${date} ${month} ${year}`;
}

const showTitle = () => {
  document.title = "Weather: " + query.value;
}

let imageUrl = ref('');
const fetchImage = () => {
  if (weather.value.weather && weather.value.weather.length > 0) {
    imageUrl.value = `https://source.unsplash.com/1080x720/?${weather.value.weather[0].main}`;
  } else {
    imageUrl.value = 'https://source.unsplash.com/1080x720/?clouds';
  }
}


onMounted(async () => {
  fetchWeather();
  fetchImage();
  try {
    let coordinates = await getCurrentLocation();
    query.value = await getCityName(coordinates.latitude, coordinates.longitude);
    showTitle();
    fetchWeather();
    fetchImage();

  } catch (error) {
    console.error(error);
  }

});


</script>


<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Courier New', Courier, monospace;
}

#app {
  /* background-image: url('https://source.unsplash.com/1080x720/?mist'); */
  background-size: cover;
  background-position: center;
  transition: 0.4s;
}

#app.warm {
  background-image: url('./assets/summer.jpg');
}

main {
  min-height: 100vh;
  padding: 25px;
  background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.25), rgba(0, 0, 0, 0.75));
}

.search-box {
  width: 100%;
  margin-bottom: 30px;
}

.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;

  color: #313131;
  font-size: 20px;
  border: none;
  appearance: none;
  outline: none;
  background: none;

  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}

.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;

}

.location-box .location {
  color: #FFF;
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}

.location-box .date {
  color: #FFF;
  font-size: 20px;
  font-weight: 300;
  font-style: italic;
  text-align: center;
}

.weather-box {
  text-align: center;
}

.weather-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color: #FFF;
  font-size: 102px;
  font-weight: 900;

  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 30px 0px;

  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-box .weather {
  color: #FFF;
  font-size: 48px;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
</style>
