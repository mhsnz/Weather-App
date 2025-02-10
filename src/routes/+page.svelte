<script lang="ts">
  import { onMount } from 'svelte';
  import { fade, fly, slide } from 'svelte/transition';

  let city = '';
  let weatherInfo: any = null;
  let forecastInfo: any = { hourly: [], daily: [] };
  let loading = false;
  let error: any = null;
  let showWeather = false;
  let isNight = false;
  let selectedDay: any = null;
  let showModal = false;
  let temperatureMessage: string = ''; // پیام بر اساس دما
  const API_KEY = '0ab98e88df7c8d0da4dde8a63121d1f3';

  // بررسی حالت شب یا روز
  function checkNightMode(sunrise: number, sunset: number) {
    const now = Math.floor(Date.now() / 1000);
    isNight = now < sunrise || now > sunset;
  }

  // باز کردن Modal و نمایش پیام بر اساس دما
  function openModal(day: any) {
    selectedDay = forecastInfo.hourly.filter((hour: any) => {
      const hourDate = new Date(hour.time).toLocaleDateString('en-US', { weekday: 'short' });
      return hourDate === day.day;
    });

    // تعیین پیام بر اساس دمای روز
    if (day.temp < 0) {
      temperatureMessage = "Bundle up, it's freezing cold!";
    } else if (day.temp >= 0 && day.temp <= 5) {
      temperatureMessage = "Don't think it's warm yet, it's still cold!";
    } else {
      temperatureMessage = "Time to look stylish without a jacket!";
    }

    showModal = true;
  }

  // بستن Modal
  function closeModal() {
    showModal = false;
    selectedDay = null;
    temperatureMessage = '';
  }

  // دریافت اطلاعات آب و هوا بر اساس مختصات جغرافیایی
  async function getWeatherByCoords(lat: number, lon: number) {
    loading = true;
    error = null;
    weatherInfo = null;
    forecastInfo = { hourly: [], daily: [] };
    showWeather = false;

    try {
      const weatherUrl = `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&units=metric&appid=${API_KEY}`;
      const weatherResponse = await fetch(weatherUrl);
      const weatherData = await weatherResponse.json();

      // بررسی زمان طلوع و غروب خورشید
      checkNightMode(weatherData.sys.sunrise, weatherData.sys.sunset);

      weatherInfo = {
        icon: getWeatherIcon(weatherData.weather[0].description),
        name: weatherData.name,
        tempCelsius: weatherData.main.temp.toFixed(1),
        description: weatherData.weather[0].description
      };

      // دریافت پیش‌بینی آب و هوا
      const forecastUrl = `https://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lon}&units=metric&appid=${API_KEY}`;
      const forecastResponse = await fetch(forecastUrl);
      const forecastData = await forecastResponse.json();

      forecastInfo.hourly = forecastData.list.slice(0, 12).map((item: any) => ({
        time: new Date(item.dt * 1000).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        temp: item.main.temp.toFixed(1),
        icon: getWeatherIcon(item.weather[0].description)
      }));

      let dailyTemps: any = {};
      let dailyIcons: any = {};

      forecastData.list.forEach((item: any) => {
        let day = new Date(item.dt * 1000).toLocaleDateString('en-US', { weekday: 'short' });
        if (!dailyTemps[day]) {
          dailyTemps[day] = [];
          dailyIcons[day] = item.weather[0].description;
        }
        dailyTemps[day].push(item.main.temp);
      });

      let daysSorted = [...Array(7).keys()].map(i => {
        return new Date(new Date().setDate(new Date().getDate() + i)).toLocaleDateString('en-US', { weekday: 'short' });
      });

      forecastInfo.daily = daysSorted.map(day => ({
        day,
        temp: (dailyTemps[day] || [0]).reduce((a: number, b: number) => a + b, 0) / (dailyTemps[day] || [1]).length,
        icon: getWeatherIcon(dailyIcons[day])
      }));

      showWeather = true;
    } catch (err: any) {
      error = `Failed to fetch weather data: ${err.message}`;
    } finally {
      loading = false;
    }
  }

  // دریافت لوکیشن کاربر
  function getUserLocation() {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          const { latitude, longitude } = position.coords;
          getWeatherByCoords(latitude, longitude);
        },
        (err) => {
          error = `Failed to get location: ${err.message}`;
        }
      );
    } else {
      error = "Geolocation is not supported by this browser.";
    }
  }

  // انتخاب آیکون بر اساس وضعیت هوا
  function getWeatherIcon(description: string) {
    if (!description) return '❓';
    if (description.includes('clear')) return '☀️';
    if (description.includes('cloud')) return '☁️';
    if (description.includes('rain')) return '🌧';
    if (description.includes('snow')) return '❄️';
    if (description.includes('storm')) return '⛈';
    return '❓';
  }

  // بارگذاری اولیه
  onMount(() => {
    getUserLocation(); // دریافت لوکیشن کاربر هنگام بارگذاری صفحه
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
    cursor: pointer;
  }

  .forecast-card p {
    margin: 5px 0;
    font-size: 14px;
  }

  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
  }

  .modal-content {
    background: var(--box-bg);
    width: 80%;
    max-width: 400px;
    border-radius: 15px;
    padding: 20px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    text-align: center;
  }

  .modal-header button {
    background: none;
    border: none;
    font-size: 24px;
    cursor: pointer;
    color: var(--text-color);
  }

  .message {
    font-size: 18px;
    font-weight: bold;
    margin-top: 20px;
  }

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
  <div class="search-box">
    <input
      type="text"
      bind:value={city}
      placeholder="Search for a city..."
    />
    <button on:click={() => getWeatherByCoords(35.6895, 51.3890)}>Search</button>
  </div>

  <div class="content">
    {#if showWeather}
      <div class="weather-box" transition:fade>
        <div class="text-4xl mb-2">{weatherInfo.icon}</div>
        <h2>{weatherInfo.name}</h2>
        <p>Temperature: {weatherInfo.tempCelsius}°C</p>
        <p>Weather: {weatherInfo.description}</p>
      </div>
    {/if}

    <div class="scrollable-forecast">
      {#each forecastInfo.hourly as hour}
        <div class="forecast-card" transition:fly={{ y: 50, duration: 500 }}>
          <div>{hour.icon}</div>
          <p>{hour.time}</p>
          <p>{hour.temp}°C</p>
        </div>
      {/each}
    </div>

    <div class="scrollable-forecast">
      {#each forecastInfo.daily as day}
        <div class="forecast-card" transition:fly={{ y: 50, duration: 500 }} on:click={() => openModal(day)}>
          <div>{day.icon}</div>
          <p>{day.day}</p>
          <p>{day.temp.toFixed(1)}°C</p>
        </div>
      {/each}
    </div>
  </div>

  {#if showModal}
    <div class="modal-overlay" on:click={closeModal}>
      <div class="modal-content" transition:fly={{ y: -50, duration: 300 }} on:click|stopPropagation>
        <button on:click={closeModal}>×</button>
        <div class="message" transition:fade>{temperatureMessage}</div>
      </div>
    </div>
  {/if}
</div>
