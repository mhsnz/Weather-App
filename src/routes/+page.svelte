<script lang="ts">
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';

  let city = '';
  let weatherInfo: any = null;
  let forecastInfo: any = null;
  let loading = false;
  let error: any = null;
  let showWeather = false;
  let suggestions: string[] = [];
  const API_KEY = '0ab98e88df7c8d0da4dde8a63121d1f3';
  let debounceTimeout: NodeJS.Timeout;

  async function getWeather(selectedCity = 'Tehran') {
    if (!selectedCity) return;
    loading = true;
    error = null;
    weatherInfo = null;
    forecastInfo = null;
    showWeather = false;
    
    try {
      await new Promise(resolve => setTimeout(resolve, 1500));

      // دریافت اطلاعات آب‌وهوا
      const weatherUrl = `https://api.openweathermap.org/data/2.5/weather?q=${selectedCity}&units=metric&appid=${API_KEY}`;
      const weatherResponse = await fetch(weatherUrl);
      const weatherData = await weatherResponse.json();

      const tempCelsius = weatherData.main.temp;
      const tempFahrenheit = tempCelsius * 9 / 5 + 32;
      const description = weatherData.weather[0].description.toLowerCase();

      let weatherIcon = '❓';
      if (description.includes('clear')) weatherIcon = '☀️';
      else if (description.includes('cloud')) weatherIcon = '☁️';
      else if (description.includes('rain')) weatherIcon = '🌧';
      else if (description.includes('snow')) weatherIcon = '❄️';
      else if (description.includes('storm')) weatherIcon = '⛈';

      weatherInfo = {
        icon: weatherIcon,
        name: weatherData.name,
        tempCelsius: tempCelsius.toFixed(1),
        tempFahrenheit: tempFahrenheit.toFixed(1),
        description: weatherData.weather[0].description
      };

      // دریافت پیش‌بینی ساعتی و روزانه
      const forecastUrl = `https://api.openweathermap.org/data/2.5/forecast?q=${selectedCity}&units=metric&appid=${API_KEY}`;
      const forecastResponse = await fetch(forecastUrl);
      const forecastData = await forecastResponse.json();

      forecastInfo = {
        hourly: forecastData.list.slice(0, 6).map(item => ({
          time: new Date(item.dt * 1000).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
          temp: item.main.temp.toFixed(1),
          icon: item.weather[0].icon
        })),
        daily: []
      };

      // تجزیه پیش‌بینی روزانه
      let dailyTemps: any = {};
      forecastData.list.forEach(item => {
        let date = new Date(item.dt * 1000).toLocaleDateString();
        if (!dailyTemps[date]) {
          dailyTemps[date] = [];
        }
        dailyTemps[date].push(item.main.temp);
      });

      forecastInfo.daily = Object.entries(dailyTemps).slice(0, 5).map(([date, temps]) => ({
        date,
        temp: (temps as number[]).reduce((a, b) => a + b, 0) / (temps as number[]).length
      }));

      setTimeout(() => {
        showWeather = true;
      }, 500);
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
    from { opacity: 0; transform: scale(0.8); }
    to { opacity: 1; transform: scale(1); }
  }

  .weather-box {
    animation: fadeIn 0.5s ease-out;
  }
</style>

<link rel="icon" href="thunder-icon.png" type="image/png">

<div class="min-h-screen flex flex-col items-center justify-center bg-gradient-to-br from-purple-900 to-indigo-700 p-4 relative">
  <div class="relative z-10 w-full max-w-md flex items-center space-x-2">
    <input
      type="text"
      bind:value={city}
      on:input={() => fetchSuggestions(city)}
      placeholder="Search for a city..."
      class="w-full px-4 py-3 rounded-full bg-white/20 text-white placeholder-white/70 outline-none"
    />
    <button
      on:click={() => getWeather(city)}
      class="px-6 py-3 rounded-full bg-orange-500 hover:bg-orange-600 transition-colors text-white font-medium"
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
    <div class="mt-6 text-white relative z-10 flex flex-col items-center">
      <div class="loading-spinner"></div>
      <p>Loading...</p>
    </div>
  {:else if error}
    <div class="mt-6 text-red-400 relative z-10">
      <p>{error}</p>
    </div>
  {:else if showWeather}
    <div class="mt-6 p-4 rounded-xl transition-all text-center text-white bg-white/20 backdrop-blur-lg relative z-10 weather-box">
      <div class="text-4xl mb-2">{weatherInfo.icon}</div>
      <h2 class="text-xl font-bold mb-2">{weatherInfo.name}</h2>
      <p class="mb-2">Temperature: {weatherInfo.tempCelsius}°C / {weatherInfo.tempFahrenheit}°F</p>
      <p>Weather: {weatherInfo.description}</p>

      <h3 class="mt-4 text-lg font-bold">Hourly Forecast</h3>
      {#each forecastInfo.hourly as hour}
        <p>{hour.time}: {hour.temp}°C</p>
      {/each}

      <h3 class="mt-4 text-lg font-bold">Daily Forecast</h3>
      {#each forecastInfo.daily as day}
        <p>{day.date}: {day.temp.toFixed(1)}°C</p>
      {/each}
    </div>
  {/if}
</div>
