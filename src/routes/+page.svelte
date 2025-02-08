<script lang="ts">
  import { onMount } from 'svelte';

  let city = '';
  let weatherInfo: any = null;
  let forecastInfo: any = { hourly: [], daily: [] };
  let loading = false;
  let error: any = null;
  let showWeather = false;
  const API_KEY = '0ab98e88df7c8d0da4dde8a63121d1f3';

  async function getWeather(selectedCity = 'Tehran') {
    if (!selectedCity) return;
    loading = true;
    error = null;
    weatherInfo = null;
    forecastInfo = { hourly: [], daily: [] };
    showWeather = false;

    try {
      await new Promise(resolve => setTimeout(resolve, 1500));

      const weatherUrl = `https://api.openweathermap.org/data/2.5/weather?q=${selectedCity}&units=metric&appid=${API_KEY}`;
      const weatherResponse = await fetch(weatherUrl);
      const weatherData = await weatherResponse.json();

      weatherInfo = {
        icon: getWeatherIcon(weatherData.weather[0].description),
        name: weatherData.name,
        tempCelsius: weatherData.main.temp.toFixed(1),
        description: weatherData.weather[0].description
      };

      const forecastUrl = `https://api.openweathermap.org/data/2.5/forecast?q=${selectedCity}&units=metric&appid=${API_KEY}`;
      const forecastResponse = await fetch(forecastUrl);
      const forecastData = await forecastResponse.json();

      forecastInfo.hourly = forecastData.list.slice(0, 24).map(item => ({
        time: new Date(item.dt * 1000).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        temp: item.main.temp.toFixed(1),
        icon: getWeatherIcon(item.weather[0].description)
      }));

      let dailyTemps: any = {};
      forecastData.list.forEach(item => {
        let day = new Date(item.dt * 1000).toLocaleDateString('en-US', { weekday: 'short' });
        if (!dailyTemps[day]) dailyTemps[day] = [];
        dailyTemps[day].push(item.main.temp);
      });

      let todayIndex = new Date().getDay();
      let daysSorted = [...Array(7).keys()].map(i => {
        return new Date(new Date().setDate(new Date().getDate() + i)).toLocaleDateString('en-US', { weekday: 'short' });
      });

      forecastInfo.daily = daysSorted.map(day => ({
        day,
        temp: (dailyTemps[day] || [0]).reduce((a, b) => a + b, 0) / (dailyTemps[day] || [1]).length,
        icon: getWeatherIcon(weatherData.weather[0].description)
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

  function getWeatherIcon(description: string) {
    if (description.includes('clear')) return '☀️';
    if (description.includes('cloud')) return '☁️';
    if (description.includes('rain')) return '🌧';
    if (description.includes('snow')) return '❄️';
    if (description.includes('storm')) return '⛈';
    return '❓';
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

  .forecast-container {
    display: flex;
    gap: 12px;
    justify-content: center;
    overflow-x: auto;
    white-space: nowrap;
    padding: 10px;
  }

  .forecast-card {
    display: flex;
    flex-direction: column;
    align-items: center;
    background: rgba(255, 255, 255, 0.2);
    padding: 8px;
    border-radius: 10px;
    min-width: 60px;
    text-align: center;
  }

  /* ریسپانسیو */
  @media (max-width: 768px) {
    .forecast-container {
      flex-wrap: wrap;
      justify-content: space-between;
    }

    .forecast-card {
      min-width: 50px;
      font-size: 12px;
    }
  }
</style>

<div class="min-h-screen flex flex-col items-center justify-center bg-gradient-to-br from-purple-900 to-indigo-700 p-4 relative">
  <div class="relative z-10 w-full max-w-md flex items-center space-x-2">
    <input
      type="text"
      bind:value={city}
      placeholder="Search for a city..."
      class="w-full px-4 py-3 rounded-full bg-white/20 text-white placeholder-white/70 outline-none"
    />
    <button on:click={() => getWeather(city)} class="px-6 py-3 rounded-full bg-orange-500 hover:bg-orange-600 transition-colors text-white font-medium">
      Confirm
    </button>
  </div>

  {#if loading}
    <div class="mt-6 text-white flex flex-col items-center">
      <div class="loading-spinner"></div>
      <p>Loading...</p>
    </div>
  {:else if error}
    <div class="mt-6 text-red-400">
      <p>{error}</p>
    </div>
  {:else if showWeather}
    <div class="mt-6 p-4 rounded-xl text-center text-white bg-white/20 backdrop-blur-lg weather-box">
      <div class="text-4xl mb-2">{weatherInfo.icon}</div>
      <h2 class="text-xl font-bold mb-2">{weatherInfo.name}</h2>
      <p class="mb-2">Temperature: {weatherInfo.tempCelsius}°C</p>
      <p>Weather: {weatherInfo.description}</p>

      <!-- نمایش پیش‌بینی ساعتی ۲۴ ساعته -->
      <h3 class="mt-4 text-lg font-bold">Hourly Forecast</h3>
      <div class="forecast-container">
        {#each forecastInfo.hourly as hour}
          <div class="forecast-card">
            <div>{hour.icon}</div>
            <div>{hour.time}</div>
            <div>{hour.temp}°C</div>
          </div>
        {/each}
      </div>

      <!-- نمایش پیش‌بینی روزانه -->
      <h3 class="mt-4 text-lg font-bold">Daily Forecast</h3>
      <div class="forecast-container">
        {#each forecastInfo.daily as day}
          <div class="forecast-card">
            <div>{day.icon}</div>
            <div>{day.day}</div>
            <div>{day.temp.toFixed(1)}°C</div>
          </div>
        {/each}
      </div>
    </div>
  {/if}
</div>
