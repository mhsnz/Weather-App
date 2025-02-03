<script lang="ts">
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';

  let city = '';
  let weatherInfo: any = null;
  let loading = false;
  let error: any = null;
  let suggestions: string[] = [];
  const API_KEY = '0ab98e88df7c8d0da4dde8a63121d1f3';

  async function getWeather(selectedCity = 'Tehran') {
    if (!selectedCity) return;
    loading = true;
    error = null;
    
    try {
      const url = `https://api.openweathermap.org/data/2.5/weather?q=${selectedCity}&units=metric&appid=${API_KEY}`;
      const response = await fetch(url);
      const data = await response.json();

      const tempCelsius = data.main.temp;
      const tempFahrenheit = tempCelsius * 9 / 5 + 32;
      const description = data.weather[0].description.toLowerCase();

      let weatherIcon = '❓';
      if (description.includes('clear')) weatherIcon = '☀️';
      else if (description.includes('cloud')) weatherIcon = '☁️';
      else if (description.includes('rain')) weatherIcon = '🌧';
      else if (description.includes('snow')) weatherIcon = '❄️';
      else if (description.includes('storm')) weatherIcon = '⛈';

      weatherInfo = {
        icon: weatherIcon,
        name: data.name,
        tempCelsius: tempCelsius.toFixed(1),
        tempFahrenheit: tempFahrenheit.toFixed(1),
        description: data.weather[0].description
      };
    } catch (err: any) {
      error = `Failed to fetch weather data: ${err.message}`;
    } finally {
      loading = false;
    }
  }

  function fetchSuggestions(query: string) {
    const cityList = ['Tehran', 'London', 'New York', 'Paris', 'Berlin', 'Tokyo', 'Sydney', 'Toronto'];
    suggestions = cityList.filter(city => city.toLowerCase().includes(query.toLowerCase())).slice(0, 5);
  }

  onMount(() => {
    getWeather();
  });
</script>

<div class="min-h-screen flex flex-col items-center justify-center bg-gradient-to-br from-blue-900 to-blue-700 p-4">
  <div class="w-full max-w-md">
    <input
      type="text"
      bind:value={city}
      on:input={() => fetchSuggestions(city)}
      placeholder="Search for a city..."
      class="w-full px-4 py-3 rounded-full bg-white/20 text-white placeholder-white/70 outline-none mb-2"
    />
    {#if suggestions.length}
      <ul class="bg-white text-black rounded-lg shadow-md overflow-hidden">
        {#each suggestions as suggestion}
          <li 
            class="px-4 py-2 hover:bg-gray-300 cursor-pointer"
            on:click={() => { city = suggestion; suggestions = []; }}
          >
            {suggestion}
          </li>
        {/each}
      </ul>
    {/if}
  </div>
  
  <button
    on:click={() => getWeather(city)}
    class="mt-4 px-6 py-3 rounded-full bg-rose-500 hover:bg-rose-600 transition-colors text-white font-medium"
  >
    Confirm
  </button>

  {#if loading}
    <div class="mt-6">
      <p>Loading...</p>
    </div>
  {:else if error}
    <div class="mt-6 text-rose-400">
      <p>{error}</p>
    </div>
  {:else if weatherInfo}
    <div class="mt-6 p-4 rounded-xl transition-all text-center text-white">
      <div class="text-4xl mb-2">{weatherInfo.icon}</div>
      <h2 class="text-xl font-bold mb-2">{weatherInfo.name}</h2>
      <p class="mb-2">Temperature: {weatherInfo.tempCelsius}°C / {weatherInfo.tempFahrenheit}°F</p>
      <p>Weather: {weatherInfo.description}</p>
    </div>
  {/if}
</div>
