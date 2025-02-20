<template>
  <div id="app" :class="{
    'warm': weather.main && weather.main.temp > 16
  }">
    <div id="background-img" :class="{
      'warm': weather.main && weather.main.temp > 16
    }">
      <div :class="{
        [weather.weather && weather.weather[0] ? weather.weather[0].main : '']: typeof weather.main !== 'undefined' && weather.main !== null
      }"></div>
      <main>
        <div class="search-box">
          <input 
            type="text" 
            class="search-bar" 
            placeholder="Поиск..." 
            name="search"
            v-model="query"
            @keypress="fetchWeather"
          />
        </div>

        <!-- Анимация загрузки -->
        <div v-if="isLoading" class="loader-block">
          <div v-if="isLoading" class="loader"></div>
        </div>
        

        <!-- Отображение погоды, если данные получены -->
        <div class="weather-wrap" v-if="typeof weather.main != 'undefined' && !isLoading">
          <div class="location-box">
            <div class="location">{{ weather.name }}, {{ weather.sys.country }}</div>
            <div class="date">{{ dateBuilder() }}</div>
          </div>
          <div class="weather-box">
            <div class="temp">{{ Math.round(weather.main.temp) }}℃</div>
            <div class="weather">{{ translateWeather(weather.weather[0].main) }}</div>
          </div>
        </div>
      </main>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'App',
    data() {
      return {
        api_key: '46ea1961936221cd4374c2a29023b8a0',
        url_base: 'https://api.openweathermap.org/data/2.5/',   
        query: '',
        weather: {},
        isLoading: false, // Переменная для управления анимацией загрузки
      }
    },
    mounted() {
      this.fetchWeather(); // Сразу вызвать при монтировании компонента
      setInterval(this.fetchWeather, 300000); // Обновление каждые 5 минут
    },
    methods: {
      async fetchWeather(e) {
        if (e && e.key !== "Enter") return;
        if (!this.query && (!this.weather.name || !this.weather.sys)) return;

        this.isLoading = true;

        try {
          const city = this.query || this.weather.name;
          const response = await fetch(`${this.url_base}weather?q=${city}&units=metric&APPID=${this.api_key}`);
          const data = await response.json();

          
          setTimeout(() => {
            this.setResults(data);
            this.isLoading = false; 
          }, 2500);
        } catch (error) {
          this.isLoading = false;
        }
      },
      setResults(results) {
        this.weather = results;
      },
      translateWeather(weatherCondition) {
        const weatherTranslations = {
          "Clear": "Ясно",
          "Clouds": "Облачно",
          "Rain": "Дождь",
          "Drizzle": "Дождь",
          "Snow": "Снег",
          // "Thunderstorm": "Гроза",
          // "Mist": "Туман",
          // "Fog": "Туман"
        };
        return weatherTranslations[weatherCondition] || weatherCondition;
      },
      dateBuilder() {
        let d = new Date();
        let months = ["Январь","Февраль", "Март", "Апрель", "Май", "Июнь", "Июль", "Август", "Сентябрь", "Октябрь", "Ноябрь", "Декабрь"];
        let days = ["Понедельник", "Вторник", "Среда", "Четверг", "Пятница", "Суббота", "Воскресенье"];

        let day = days[d.getDay()];
        let date = d.getDate();
        let month = months[d.getMonth()];
        let year = d.getFullYear();

        return `${day} ${date} ${month} ${year}`;
      }
    }
  }
</script>

<style lang="scss" scoped>
// === Переменные и миксины ===
@mixin fontSizeWeight($fs, $fw) {
  font-size: $fs;
  font-weight: $fw;
}

// === Общие стили ===
#app {
  background-color: #8398a1;
  &.warm {
    background-color: #a13b84;
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
  }
}

#background-img {
  background-image: url("./assets/image/cold-bg.png");
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
  transition: 0.6s;
  &.warm {
    background-image: url("./assets/image/warm-bg.png");
  }
}

// === Блок загрузчика ===
.loader-block {
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  overflow: hidden;
}

.loader {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 40%;
  aspect-ratio: 1 / 1;
  &::before,
  &::after {
    content: '';
    position: absolute;
    border-radius: 50%;
    animation-duration: 1.8s;
    animation-iteration-count: infinite;
    animation-timing-function: ease-in-out;
    filter: drop-shadow(0 0 11px rgba(255, 255, 255, 0.75));
  }
  &::before {
    width: 100%;
    height: 100%;
    box-shadow: inset 0 0 0 17px #fff;
    animation-name: pulsA;
  }
  &::after {
    width: calc(100% - 34px);
    height: calc(100% - 34px);
    box-shadow: 0 0 0 0 #fff;
    animation-name: pulsB;
  }
}

// === Анимации загрузчика ===
@keyframes pulsA {
  0% { box-shadow: inset 0 0 0 17px #fff; opacity: 1; }
  50%, 100% { box-shadow: inset 0 0 0 0 #fff; opacity: 0; }
}

@keyframes pulsB {
  0%, 50% { box-shadow: 0 0 0 0 #fff; opacity: 0; }
  100% { box-shadow: 0 0 0 17px #fff; opacity: 1; }
}

// === Блок погоды ===
main {
  position: relative;
  z-index: 2;
  min-height: 100vh;
  padding: 1.6rem;
  background-image: linear-gradient(to bottom, rgba(#000, 0.25), rgba(#000, 0.75));
  .search-box {
    width: 100%;
    margin-bottom: 1.7rem;
    .search-bar {
      display: block;
      width: 100%;
      padding: 1rem;
      color: #313131;
      font-size: 1.25rem;
      appearance: none;
      border: none;
      outline: none;
      background: none;
      box-shadow: 0 0 0.5rem rgba(#000, 0.25);
      background-color: rgba(#fff, 0.5);
      border-radius: 0 1rem;
      transition: 0.4s;
      &:focus {
        box-shadow: 0 0 1rem rgba(#000, 0.25);
        background-color: rgba(#fff, 0.75);
        border-radius: 1rem 0;
      }
    }
  }
  .weather-wrap {
    color: #fff;
    .location-box {
      text-align: center;
      .location { @include fontSizeWeight(2rem, 500); text-shadow: 0.1rem 0.2rem rgba(#000, 0.25); }
      .date { @include fontSizeWeight(1.4rem, 300); font-style: italic; }
    }
    .weather-box {
      text-align: center;
      .temp {
        display: inline-block;
        padding: 0.55rem 1.8rem;
        @include fontSizeWeight(6.5rem, 900);
        text-shadow: 3px 6px rgba(#000, 0.25);
        background-color: rgba(#fff, 0.25);
        border-radius: 1rem;
        margin: 2rem 0;
        box-shadow: 0.2rem 0.4rem rgba(#000, 0.25);
      }
      .weather {
        @include fontSizeWeight(3rem, 700);
        font-style: italic;
        text-shadow: 0.2rem 0.4rem rgba(#000, 0.25);
      }
    }
  }
}

// === Анимации погоды ===
.Clear {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url("./assets/animation/birds.gif");
  background-size: 30%;
  background-repeat: no-repeat;
  animation: clear 7.5s linear infinite;
  @media (min-width: 800px) { background-size: 10%; }
}

@keyframes clear {
  0% { background-position: 200% 30%; opacity: 0.1; }
  50% { background-position: 50% 30%; opacity: 0.55; }
  100% { background-position: -100% 30%; opacity: 0.1; }
}

// === Другие анимации (Облака, Дождь, Туман) ===
.Clouds {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url("./assets/animation/cloud.png");
  background-size: 50%;
  background-repeat: repeat-x;
  animation: cloud 30s linear infinite;
  z-index: 1;
}

@keyframes cloud {
  0% { background-position: -100% 10%; }
  50% { background-position: 50% 10%; }
  100% { background-position: 200% 10%; }
}

// === Адаптивность ===
@media (max-width: 750px) {
  .Mist, .Fog {
    background-size: 500%;
    animation-duration: 30s;
  }
}
</style>
