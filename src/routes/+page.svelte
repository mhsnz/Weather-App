<script lang="ts">
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';

  let city = '';
  let weatherInfo: any = null;
  let loading = false;
  let error: any = null;
  let suggestions: string[] = [];
  const API_KEY = '0ab98e88df7c8d0da4dde8a63121d1f3';
  let debounceTimeout: NodeJS.Timeout;

  async function getWeather(selectedCity = 'Tehran') {
    if (!selectedCity) return;
    loading = true;
    error = null;
    weatherInfo = null;
    
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
    clearTimeout(debounceTimeout);
    debounceTimeout = setTimeout(() => {
      const cityList = ['Tehran', 'London', 'New York', 'Paris', 'Berlin', 'Tokyo', 'Sydney', 'Toronto', 'Madrid', 'Dubai', 'Beijing', 'Moscow', 'Delhi', 'Seoul', 'Bangkok', 'Istanbul', 'Rome'];
      suggestions = cityList
        .filter(city => city.toLowerCase().includes(query.toLowerCase()))
        .slice(0, 5);
    }, 300);
  }

  onMount(() => {
    getWeather();
  });
</script>

<style>
  @keyframes fadeIn {
    from { opacity: 0; transform: scale(0.9); }
    to { opacity: 1; transform: scale(1); }
  }

  .loading-animation {
    animation: fadeIn 0.5s ease-in-out infinite alternate;
  }

  .search-bar {
    transition: all 0.2s ease-in-out;
  }
  .search-bar:focus {
    transform: scale(1.05);
    background-color: rgba(255, 255, 255, 0.3);
    box-shadow: 0px 0px 10px rgba(255, 255, 255, 0.5);
  }
  .confirm-button {
    transition: all 0.2s ease-in-out;
  }
  .confirm-button:hover {
    transform: scale(1.1);
    background-color: #ff7f50;
    box-shadow: 0px 0px 10px rgba(255, 127, 80, 0.7);
  }
  .suggestions-box {
    background: rgba(255, 255, 255, 0.2);
    backdrop-filter: blur(10px);
    border-radius: 10px;
    transition: opacity 0.2s ease-in-out;
  }
</style>

<div class="min-h-screen flex flex-col items-center justify-center bg-gradient-to-br from-purple-900 to-indigo-700 p-4 relative">
  <div class="absolute inset-0 bg-[url('https://source.unsplash.com/random/1920x1080/?nature')] bg-cover bg-center opacity-30"></div>
  <div class="relative z-10 w-full max-w-md flex items-center space-x-2">
    <input
      type="text"
      bind:value={city}
      on:input={() => fetchSuggestions(city)}
      placeholder="Search for a city..."
      class="w-full px-4 py-3 rounded-full bg-white/20 text-white placeholder-white/70 outline-none search-bar"
    />
    <button
      on:click={() => getWeather(city)}
      class="px-6 py-3 rounded-full bg-orange-500 hover:bg-orange-600 transition-colors text-white font-medium confirm-button"
    >
      Confirm
    </button>
  </div>
  {#if suggestions.length}
    <ul class="suggestions-box text-white rounded-lg shadow-md overflow-hidden w-full max-w-md mt-2 relative z-10">
      {#each suggestions as suggestion}
        <li 
          class="px-4 py-2 hover:bg-white/30 cursor-pointer"
          on:click={() => { city = suggestion; suggestions = []; }}
        >
          {suggestion}
        </li>
      {/each}
    </ul>
  {/if}

  {#if loading}
    <div class="mt-6 text-white relative z-10 loading-animation">
      <p>Loading...</p>
    </div>
  {:else if error}
    <div class="mt-6 text-red-400 relative z-10">
      <p>{error}</p>
    </div>
  {:else if weatherInfo}
    <div class="mt-6 p-4 rounded-xl transition-all text-center text-white bg-white/20 backdrop-blur-lg relative z-10">
      <div class="text-4xl mb-2">{weatherInfo.icon}</div>
      <h2 class="text-xl font-bold mb-2">{weatherInfo.name}</h2>
      <p class="mb-2">Temperature: {weatherInfo.tempCelsius}°C / {weatherInfo.tempFahrenheit}°F</p>
      <p>Weather: {weatherInfo.description}</p>
    </div>
  {/if}
</div>
