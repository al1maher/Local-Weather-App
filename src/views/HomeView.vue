<template>

  <main class="container text-white">

    <div class="relative pt-4 mb-8">

      <input @input="getSearchResults" v-model="searchQuery" autocomplete="off" autosave="off" type="text" placeholder="Search for a city!" class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0.px_1px_0_0_#004E71]">

      <transition name="fade">
      <ul v-if="mapboxSearchResults"
      class="absolute bg-weather-primary  text-white shadow-2xl rounded  py-2 px-1 w-full top-[66px]">
        <p v-if="searchError">Sorry, something went wrong. Please try again!</p>
        <p v-if="!searchError && mapboxSearchResults.length === 0">
          No matched search results for your query! Try a different input.
        </p>

        <template v-else>
          <li v-for="searchResult in mapboxSearchResults" :key="searchResult.id"
            class="py-2 cursor-pointer border-b-2 border-weather-secondary border-opacity-30 hover:bg-weather-secondary" @click="previewCity(searchResult)">
            {{ searchResult.place_name }}
          </li>
        </template>

      </ul>
    </transition>

    </div>

    <div class="flex flex-col gap-4">
       <Suspense>
         <CityList />
         <template #fallback>
          <CityCardSkeleton />
         </template>
       </Suspense>
    </div>

  </main>

</template>


<script setup>
  import { ref } from 'vue';
  import axios from 'axios';
  import { useRouter } from 'vue-router';
  import CityList from "../components/CityList.vue"
  import CityCardSkeleton from "../components/CityCardSkeleton.vue"

  const router = useRouter();
  const searchQuery = ref('');
  const queryTimeout = ref(null);
  const mapboxApiKey = 'pk.eyJ1IjoiYWxpbWFoZXIxIiwiYSI6ImNsbjNzNG1ucjBxMGkybHBocmh3bXgzY2MifQ.NPBtwhE5AodWv6ZKTvLUsQ';
  const mapboxSearchResults = ref(null);
  const searchError = ref(null);

  const getSearchResults = () => {
    clearTimeout(queryTimeout.value);
    queryTimeout.value = setTimeout(async() => {
      if (searchQuery.value != '') {
        axios.get(`https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxApiKey}&types=place`)
        .then(response => mapboxSearchResults.value = response.data.features )
        .catch(err => searchError.value = err);

        return;
      }
      
      mapboxSearchResults.value = null;

    }, 300)
  };

  const previewCity = (searchResult) => {
    console.log(searchResult);
    const [city, state] = searchResult.place_name.split(", ");
    console.log([city,state]);
    router.push({
      name: 'cityView',
      params: {state: state, city: city},
      query: {
        lat: searchResult.geometry.coordinates[1],
        lng: searchResult.geometry.coordinates[0],
        preview: true,

      }
    })
  }
</script>

<style scoped>
  .fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>


