<template>
  <div class="flex flex-col">

    <!--Banner-->
    <div v-if="Route.query.preview" class="text-white p-4 bg-weather-secondary/30 w-full  text-center">
      <p>You're currently previewing this city, click the "+" icon to start tracking this city.</p>
    </div>
 

  <!--Weather Overview-->
  <div class="flex flex-col items-center text-white py-12" v-if="weatherData">
    
    <h1 class="text-4xl mb-2">{{ Route.params.city }}</h1>

    <p class="text-sm mb-12">
      {{ 
        new Date(weatherData.currentTime).toLocaleDateString(
          "en-us",
          {
            weekday: "short",
            day: "2-digit",
            month: "long"
          }
        )
      }}

      {{ 
        new Date(weatherData.currentTime).toLocaleTimeString(
          "en-us",
          {
            hour: "2-digit",
            minute: "2-digit"
          }
        )
      }}
    </p>

    <p class="text-8xl mb-8">
      {{ Math.round(weatherData.current.temp) }}&deg;C
    </p>

    <p>
      Feels like:
      {{ Math.round(weatherData.current.feels_like) }}&deg;C
    </p>

    <p class="capitalize">
      {{ weatherData.current.weather[0].description }}
    </p>

    <img
      class="w-[150px] h-auto"
      :src="`https://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`" >
  </div>


  <hr class="border-white border-opacity-30 border-w-full">


  <!--Hourly Weather-->
  <div class="max-w-screen-md w-full py-12 self-center">
    <div class="mx-8 text-white">
      <h2 class="mb-8 text-2xl">Hourly Weather</h2>
      

      <div class="flex gap-10 overflow-x-clip hover:overflow-x-scroll">
        <template v-if="weatherData && weatherData.hourly">
          <div v-for="hourData in weatherData.hourly" :key="hourData.dt" class="flex flex-col gap-4 items-center hover:bg-black/5">
            
          <p class="whitespace-nowrap text-md">
            {{ 
              new Date(hourData.currentTime)
                .toLocaleTimeString('en-us', {hour: 'numeric'})
            }}
          </p>

          <img
            class="w-auto h-[50px] object-cover"
            :src="`https://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`"
          >

          <p class="text-xl">
            {{ Math.round(hourData.temp) }}&deg;C
          </p>


        </div>
        </template>
      </div>
    </div>
  </div>

  <hr class="border-white border-opacity-30 border-w-full">

  <!--Weekly Weather-->
  <div class="max-w-screen-md w-full py-12 self-center">
    <div class="mx-8 text-white">
      <h2 class="mb-8 text-2xl">7 Day Forecast</h2>

      <template v-if="weatherData && weatherData.daily">
        <div v-for="day in weatherData.daily" :key="day.dt" class="flex items-center hover:bg-black/10">

          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString('en-us', {weekday: 'long'})
            }}
          </p>

          <img
            class="w-[50px] h-[50px] object-cover"
            :src="`https://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`"
          >

          <div class="flex flex-1 gap-2 justify-end">
            <p>H: {{ Math.round(day.temp.max) }}</p>
            <p>L: {{ Math.round(day.temp.min) }}</p>
          </div>

        </div>
      </template>

    </div>
  </div>


  <hr class="border-white border-opacity-30 border-w-full">

  <!--Remove City-->
  <div class="flex items-center self-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500" @click="removeCity">
    <i class="fa-solid fa-trash"></i>
    <p>Remove City</p>
  </div>


  </div>
</template>



<script setup>

  import axios from 'axios';
  import { useRoute, useRouter } from 'vue-router';
  import { ref, onMounted } from 'vue';
  import moment from 'moment-timezone'; // import moment-timezone

  const Route = useRoute();
  const weatherData = ref(null);

  // Use a method to fetch the weather data
  const fetchWeatherData = async () => {
    const lat = Route.query.lat;
    const lng = Route.query.lng;
    const key = '5a04735b120623f8fd983848a5e18677';

    try {
      const response = await axios.get(`https://api.openweathermap.org/data/3.0/onecall?lat=${lat}&lon=${lng}&appid=${key}&units=metric`);
      weatherData.value = response.data;

      // Calculate the current and hourly time offsets
      // Use moment.tz() to convert the UTC timestamps to the desired timezone
      const timezone = 'Africa/Cairo'; // specify the timezone you want
      weatherData.value.currentTime = moment.tz(weatherData.value.current.dt * 1000, timezone).format(); // format the date and time as you wish
      weatherData.value.hourly.forEach(hour => {
        hour.currentTime = moment.tz(hour.dt * 1000, timezone).format();
      });

      await new Promise((res) => setTimeout(res, 1000));

      return weatherData.data;

    } catch (error) {
      console.error(error);
    }
    
  };

  // Call the fetchWeatherData method when the component is mounted
  await fetchWeatherData();

  const router = useRouter();
  const removeCity = () => {
    const cities = JSON.parse(localStorage.getItem('savedCities'));
    const updatedCities = cities.filter((city) => {
      return city.id !== Route.query.id;
    });
    localStorage.setItem('savedCities', JSON.stringify(updatedCities));
    
    router.push({
      name: 'home'
    })
  }


</script>


