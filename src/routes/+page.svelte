<script lang="ts">
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';

  let city = '';
  let weatherInfo: any = null;
  let forecast: any = null;
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
    forecast = null;
    showWeather = false;
    
    try {
      await new Promise(resolve => setTimeout(resolve, 1500));
      
      // Fetch current weather
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

      // Fetch weather forecast
      const forecastUrl = `https://api.openweathermap.org/data/2.5/forecast?q=${selectedCity}&units=metric&appid=${API_KEY}`;
      const forecastResponse = await fetch(forecastUrl);
      const forecastData = await forecastResponse.json();
      
      forecast = forecastData.list.slice(0, 5).map(entry => ({
        time: new Date(entry.dt * 1000).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        tempCelsius: entry.main.temp.toFixed(1),
        tempFahrenheit: (entry.main.temp * 9 / 5 + 32).toFixed(1),
        description: entry.weather[0].description,
        icon: entry.weather[0].icon
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

  onMount(() => {
    getWeather();
  });
</script>

<div class="mt-6 p-4 rounded-xl transition-all text-center text-white bg-white/20 backdrop-blur-lg relative z-10 weather-box">
  <div class="text-4xl mb-2">{weatherInfo.icon}</div>
  <h2 class="text-xl font-bold mb-2">{weatherInfo.name}</h2>
  <p class="mb-2">Temperature: {weatherInfo.tempCelsius}°C / {weatherInfo.tempFahrenheit}°F</p>
  <p>Weather: {weatherInfo.description}</p>
</div>

{#if forecast}
  <div class="mt-4 p-4 bg-white/20 rounded-lg text-white text-center">
    <h3 class="text-lg font-bold mb-2">Upcoming Forecast</h3>
    <ul>
      {#each forecast as entry}
        <li class="mb-2">
          <span class="font-bold">{entry.time}:</span> {entry.tempCelsius}°C / {entry.tempFahrenheit}°F - {entry.description}
        </li>
      {/each}
    </ul>
  </div>
{/if}
