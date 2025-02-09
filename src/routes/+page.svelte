<script lang="ts">
  import { onMount } from 'svelte';

  let city = '';
  let weatherInfo: any = null;
  let forecastInfo: any = { hourly: [], daily: [] };
  let loading = false;
  let error: any = null;
  let showWeather = false;
  let isNight = false; // حالت شب یا روز
  const API_KEY = '0ab98e88df7c8d0da4dde8a63121d1f3';

  async function getWeather(selectedCity = 'Tehran') {
    if (!selectedCity) return;
    loading = true;
    error = null;
    weatherInfo = null;
    forecastInfo = { hourly: [], daily: [] };
    showWeather = false;

    try {
      const weatherUrl = `https://api.openweathermap.org/data/2.5/weather?q=${selectedCity}&units=metric&appid=${API_KEY}`;
      const weatherResponse = await fetch(weatherUrl);
      const weatherData = await weatherResponse.json();

      // بررسی زمان طلوع و غروب خورشید
      const now = Math.floor(Date.now() / 1000); // زمان فعلی به ثانیه
      const sunrise = weatherData.sys.sunrise; // زمان طلوع خورشید
      const sunset = weatherData.sys.sunset; // زمان غروب خورشید
      isNight = now < sunrise || now > sunset; // اگر قبل از طلوع یا بعد از غروب باشد، شب است

      weatherInfo = {
        icon: getWeatherIcon(weatherData.weather[0].description),
        name: weatherData.name,
        tempCelsius: weatherData.main.temp.toFixed(1),
        description: weatherData.weather[0].description
      };

      const forecastUrl = `https://api.openweathermap.org/data/2.5/forecast?q=${selectedCity}&units=metric&appid=${API_KEY}`;
      const forecastResponse = await fetch(forecastUrl);
      const forecastData = await forecastResponse.json();

      forecastInfo.hourly = forecastData.list.slice(0, 12).map(item => ({
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

      let daysSorted = [...Array(7).keys()].map(i => {
        return new Date(new Date().setDate(new Date().getDate() + i)).toLocaleDateString('en-US', { weekday: 'short' });
      });

      forecastInfo.daily = daysSorted.map(day => ({
        day,
        temp: (dailyTemps[day] || [0]).reduce((a, b) => a + b, 0) / (dailyTemps[day] || [1]).length,
        icon: getWeatherIcon(weatherData.weather[0].description)
      }));

      showWeather = true;
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
  :global(body) {
    margin: 0;
    font-family: 'Arial', sans-serif;
    color: #333;
    transition: background 0.3s ease, color 0.3s ease;
  }

  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    padding: 20px;
    background: var(--background);
    color: var(--text-color);
  }

  .search-box {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 10px;
    margin-bottom: 20px;
  }

  input {
    padding: 10px 15px;
    border: none;
    border-radius: 25px;
    outline: none;
    font-size: 16px;
    background: var(--input-bg);
    color: var(--text-color);
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  button {
    padding: 10px 20px;
    border: none;
    border-radius: 25px;
    background: var(--button-bg);
    color: var(--button-text);
    font-size: 16px;
    cursor: pointer;
    transition: background 0.3s ease;
  }

  button:hover {
    background: var(--button-hover-bg);
  }

  .content {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
    width: 100%;
    max-width: 800px;
  }

  .weather-box {
    text-align: center;
    padding: 20px;
    border-radius: 15px;
    background: var(--box-bg);
    backdrop-filter: blur(10px);
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    width: 100%;
    max-width: 300px;
  }

  .weather-box h2 {
    margin: 0;
    font-size: 24px;
    font-weight: bold;
  }

  .weather-box p {
    margin: 5px 0;
    font-size: 18px;
  }

  .scrollable-forecast {
    display: flex;
    overflow-x: auto;
    gap: 10px;
    padding: 10px;
    width: 100%;
    max-width: 600px;
    scrollbar-width: thin;
    scrollbar-color: var(--scrollbar-thumb) var(--scrollbar-track);
  }

  .scrollable-forecast::-webkit-scrollbar {
    height: 8px;
  }

  .scrollable-forecast::-webkit-scrollbar-thumb {
    background: var(--scrollbar-thumb);
    border-radius: 4px;
  }

  .scrollable-forecast::-webkit-scrollbar-track {
    background: var(--scrollbar-track);
  }

  .forecast-card {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 15px;
    border-radius: 15px;
    background: var(--box-bg);
    backdrop-filter: blur(10px);
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    min-width: 100px;
    flex-shrink: 0;
  }

  .forecast-card p {
    margin: 5px 0;
    font-size: 14px;
  }

  /* متغیرهای حالت لایت مود */
  :global(.light-mode) {
    --background: linear-gradient(135deg, #f5f7fa, #c3cfe2);
    --text-color: #333;
    --input-bg: rgba(255, 255, 255, 0.8);
    --button-bg: #ff6f61;
    --button-text: white;
    --button-hover-bg: #ff3b2f;
    --box-bg: rgba(255, 255, 255, 0.8);
    --scrollbar-thumb: #ff6f61;
    --scrollbar-track: #f5f7fa;
  }

  /* متغیرهای حالت دارک مود */
  :global(.dark-mode) {
    --background: linear-gradient(135deg, #1e1e2f, #2a2a40);
    --text-color: #fff;
    --input-bg: rgba(255, 255, 255, 0.1);
    --button-bg: #ff6f61;
    --button-text: white;
    --button-hover-bg: #ff3b2f;
    --box-bg: rgba(255, 255, 255, 0.1);
    --scrollbar-thumb: #ff6f61;
    --scrollbar-track: #1e1e2f;
  }
</style>

<div class="container" class:light-mode={!isNight} class:dark-mode={isNight}>
  <!-- سرچ بار -->
  <div class="search-box">
    <input
      type="text"
      bind:value={city}
      placeholder="Search for a city..."
    />
    <button on:click={() => getWeather(city)}>Search</button>
  </div>

  <!-- محتوا در وسط صفحه -->
  <div class="content">
    <!-- باکس نمایش دما -->
    {#if showWeather}
      <div class="weather-box">
        <div class="text-4xl mb-2">{weatherInfo.icon}</div>
        <h2>{weatherInfo.name}</h2>
        <p>Temperature: {weatherInfo.tempCelsius}°C</p>
        <p>Weather: {weatherInfo.description}</p>
      </div>
    {/if}

    <!-- باکس پیش‌بینی ساعتی با قابلیت اسکرول -->
    <div class="scrollable-forecast">
      {#each forecastInfo.hourly as hour}
        <div class="forecast-card">
          <div>{hour.icon}</div>
          <p>{hour.time}</p>
          <p>{hour.temp}°C</p>
        </div>
      {/each}
    </div>

    <!-- باکس پیش‌بینی روزانه با قابلیت اسکرول -->
    <div class="scrollable-forecast">
      {#each forecastInfo.daily as day}
        <div class="forecast-card">
          <div>{day.icon}</div>
          <p>{day.day}</p>
          <p>{day.temp.toFixed(1)}°C</p>
        </div>
      {/each}
    </div>
  </div>
</div>
