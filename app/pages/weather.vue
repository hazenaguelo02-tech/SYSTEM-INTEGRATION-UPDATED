
<template>
  <div class="page">
    <ClientOnly>
      <v-card class="card pa-5">

        <div class="d-flex justify-space-between align-center">
          <span>{{ time }}</span>
          <v-btn icon variant="text" color="white"
            :loading="loading" @click="getWeather()">
            <v-icon>mdi-refresh</v-icon>
          </v-btn>
        </div>

        <div class="text-center pa-5">
          <h3>{{ city }}</h3>
          <p>Current Temperature</p>
          <h1 class="text-h2 my-3">{{ temp }}°C</h1>

          <v-icon size="80" color="light-blue">
            {{ icon }}
          </v-icon>

          <p>{{ description }}</p>
          <p>Feels Like: {{ feelsLike }}°C</p>
          <p>Humidity: {{ humidity }}%</p>
          <p>Wind: {{ wind }} km/h</p>

          <p v-if="loading">Loading weather...</p>
          <p v-if="error" class="error">{{ error }}</p>
        </div>

        <div class="d-flex justify-space-around flex-wrap ga-2">
          <v-btn class="weather-btn" @click="period = 'Hourly'">
            Hourly
          </v-btn>
          <v-btn class="weather-btn" @click="period = 'Daily'">
            Daily
          </v-btn>
          <v-btn class="weather-btn" @click="period = 'Weekly'">
            Weekly
          </v-btn>
          <v-btn class="weather-btn" @click="addCity">
            Add City
          </v-btn>
        </div>

        <p class="text-center mt-4">{{ period }} Forecast</p>

        <div v-if="period === 'Hourly'" class="forecast">
          <div v-for="h in hours" :key="h.date" class="forecast-item">
            <p>{{ h.time }}</p>
            <v-icon color="light-blue">{{ h.icon }}</v-icon>
            <p>{{ h.temp }}°</p>
          </div>
        </div>

        <div v-else class="forecast">
          <div v-for="d in daysToShow" :key="d.date" class="forecast-item">
            <p>{{ d.day }}</p>
            <v-icon color="light-blue">{{ d.icon }}</v-icon>
            <p>{{ d.temp }}°</p>
            <small>{{ d.min }}° / {{ d.max }}°</small>
          </div>
        </div>

      </v-card>
    </ClientOnly>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const city = ref('SAN FERNANDO')
const time = ref('--')
const temp = ref('--')
const feelsLike = ref('--')
const humidity = ref('--')
const wind = ref('--')
const description = ref('Loading...')
const icon = ref('mdi-weather-cloudy')
const loading = ref(false)
const error = ref('')
const period = ref('Hourly')
const hours = ref([])
const days = ref([])

const daysToShow = computed(() => {
  return period.value === 'Daily'
    ? days.value.slice(0, 1)
    : days.value
})

function weatherText(code) {
  if (code === 0) return 'Clear Sky'
  if (code === 1) return 'Mainly Clear'
  if (code === 2) return 'Partly Cloudy'
  if (code === 3) return 'Overcast'
  if ([45, 48].includes(code)) return 'Foggy'
  if ([51, 53, 55, 56, 57].includes(code)) return 'Drizzle'
  if ([61, 63, 65, 66, 67].includes(code)) return 'Rain'
  if ([71, 73, 75, 77].includes(code)) return 'Snow'
  if ([80, 81, 82].includes(code)) return 'Rain Showers'
  if ([85, 86].includes(code)) return 'Snow Showers'
  if ([95, 96, 99].includes(code)) return 'Thunderstorm'
  return 'Partly Cloudy'
}

function weatherIcon(code) {
  if (code === 0) return 'mdi-weather-sunny'
  if ([1, 2].includes(code)) return 'mdi-weather-partly-cloudy'
  if (code === 3) return 'mdi-weather-cloudy'
  if ([45, 48].includes(code)) return 'mdi-weather-fog'
  if ([51, 53, 55, 56, 57, 61, 63, 65, 66, 67, 80, 81, 82].includes(code))
    return 'mdi-weather-rainy'
  if ([71, 73, 75, 77, 85, 86].includes(code))
    return 'mdi-weather-snowy'
  if ([95, 96, 99].includes(code))
    return 'mdi-weather-lightning-rainy'
  return 'mdi-weather-cloudy'
}

async function getWeather(place = 'San Fernando, Pampanga') {
  loading.value = true
  error.value = ''

  try {
    const geo = await $fetch(
      'https://geocoding-api.open-meteo.com/v1/search',
      {
        query: {
          name: place,
          count: 20,
          language: 'en',
          format: 'json'
        }
      }
    )

    const location = geo.results?.find(
      item => item.country_code === 'PH'
    )

    if (!location) {
      throw new Error('City not found. Please try again.')
    }

    const data = await $fetch(
      'https://api.open-meteo.com/v1/forecast',
      {
        query: {
          latitude: location.latitude,
          longitude: location.longitude,
          current: 'temperature_2m,relative_humidity_2m,apparent_temperature,weather_code,wind_speed_10m',
          hourly: 'temperature_2m,weather_code',
          daily: 'weather_code,temperature_2m_max,temperature_2m_min',
          timezone: 'auto',
          forecast_days: 7
        }
      }
    )

    const now = data.current

    city.value = location.name.toUpperCase()
    temp.value = Math.round(now.temperature_2m)
    feelsLike.value = Math.round(now.apparent_temperature)
    humidity.value = now.relative_humidity_2m
    wind.value = Math.round(now.wind_speed_10m)
    description.value = weatherText(now.weather_code)
    icon.value = weatherIcon(now.weather_code)

    time.value = new Date(now.time).toLocaleTimeString(
      'en-PH',
      { hour: '2-digit', minute: '2-digit', hour12: true }
    )

    hours.value = data.hourly.time
      .map((t, i) => ({
        date: t,
        time: new Date(t).toLocaleTimeString('en-PH', {
          hour: '2-digit',
          hour12: true
        }),
        temp: Math.round(data.hourly.temperature_2m[i]),
        icon: weatherIcon(data.hourly.weather_code[i])
      }))
      .filter(h => h.date > now.time)
      .slice(0, 5)

    days.value = data.daily.time.map((date, i) => ({
      date,
      day: new Date(date + 'T12:00:00').toLocaleDateString(
        'en-PH',
        { weekday: 'short' }
      ),
      temp: Math.round(data.daily.temperature_2m_max[i]),
      min: Math.round(data.daily.temperature_2m_min[i]),
      max: Math.round(data.daily.temperature_2m_max[i]),
      icon: weatherIcon(data.daily.weather_code[i])
    }))

  } catch (e) {
    console.error('Weather Error:', e)
    error.value = e.message || 'Unable to get weather data.'
  } finally {
    loading.value = false
  }
}

function addCity() {
  const place = window.prompt('Enter city name:')

  if (place?.trim()) {
    period.value = 'Hourly'
    getWeather(place.trim())
  }
}

onMounted(() => {
  getWeather()
})
</script>

<style>
.page {
  min-height: 100vh;
  display: grid;
  place-items: center;
  padding: 20px;
  background: linear-gradient(135deg, #11013a, #521ab4);
}

.card {
  width: 100%;
  max-width: 400px;
  min-height: 600px;
  border-radius: 25px;
  color: white;
  background: rgba(255,255,255,0.12) !important;
  border: 1px solid rgba(255,255,255,0.25);
  backdrop-filter: blur(15px);
  -webkit-backdrop-filter: blur(15px);
  box-shadow: 0 8px 32px rgba(0,0,0,0.25);
}

.weather-btn {
  min-width: 75px;
  padding: 0 10px;
  color: white !important;
  background: rgba(255,255,255,0.12) !important;
  border: 1px solid rgba(255,255,255,0.3);
}

.forecast {
  display: flex;
  justify-content: space-around;
  gap: 8px;
  margin-top: 24px;
  text-align: center;
  flex-wrap: wrap;
}

.forecast-item {
  min-width: 48px;
}

.error {
  color: #ffb4b4;
}
</style>






