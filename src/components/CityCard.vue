<template>
  <div  v-if="city.weather" class="flex py-6 px-3 bg-weather-secondary rounded-lg shadow-2xl cursor-pointer hover:bg-weather-primary">
      <div class="flex flex-col flex-1">
        <h2 class="text-3xl">{{ city.city }}</h2>
        <h3>{{ city.state }}</h3>
      </div>
      <div class="flex flex-col gap-2">
        <p class="text-3xl self-end">
          {{ Math.round(city.weather.main.temp) }}&deg;C
        </p>
        <div class="flex gap-2">
          <span class="text-xs">
            H: {{ Math.round(city.weather.main.temp_max) }}
          </span>
          <span class="text-xs">
            L: {{ Math.round(city.weather.main.temp_min) }}
          </span>
        </div>
      </div>
  </div>
</template>

<script setup>
  defineProps({
    city: {
      type: Object,
      default: () => ({ weather: {}})
    },
  });
</script>

