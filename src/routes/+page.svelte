<!-- WeatherApp.svelte -->
<script lang="ts">
    import { onMount } from 'svelte';
  
    let country = '';
    let weatherInfo: any = null;
    let loading = false;
    let error: any = null;
  
    const API_KEY = '0ab98e88df7c8d0da4dde8a63121d1f3';
  
    async function getWeather(selectedCountry = 'Tehran') {
      if (!selectedCountry) return;
      
      loading = true;
      error = null;
  
      try {
        const url = `https://api.openweathermap.org/data/2.5/weather?q=${selectedCountry}&units=metric&appid=${API_KEY}`;
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
  
    onMount(() => {
      getWeather();
    });
  </script>
  
  <div class="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-900 to-blue-700 p-4">
    <div class="bg-white/10 backdrop-blur-lg p-8 rounded-2xl w-full max-w-md text-center text-white">
      <h1 class="text-2xl font-bold mb-6">Weather App</h1>
      
      <input
        type="text"
        bind:value={country}
        placeholder="Enter a country name"
        class="w-full px-4 py-3 rounded-full bg-white/20 text-white placeholder-white/70 outline-none mb-4"
      />
      
      <button
        on:click={() => getWeather(country)}
        class="w-full px-4 py-3 rounded-full bg-rose-500 hover:bg-rose-600 transition-colors text-white font-medium"
      >
        Get Weather
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
        <div class="mt-6 p-4 rounded-xl transition-all">
          <div class="text-4xl mb-2">{weatherInfo.icon}</div>
          <h2 class="text-xl font-bold mb-2">{weatherInfo.name}</h2>
          <p class="mb-2">Temperature: {weatherInfo.tempCelsius}°C / {weatherInfo.tempFahrenheit}°F</p>
          <p>Weather: {weatherInfo.description}</p>
        </div>
      {/if}
    </div>
  </div>