<template>
  <div class="weather" v-if="weatherData.adCode.city && weatherData.weather.weather">
    <span>{{ weatherData.adCode.city }}&nbsp;</span>
    <span>{{ weatherData.weather.weather }}&nbsp;</span>
    <span>{{ weatherData.weather.temperature }}℃</span>
    <span class="sm-hidden">
      &nbsp;{{
        weatherData.weather.winddirection?.endsWith("风")
          ? weatherData.weather.winddirection
          : weatherData.weather.winddirection + "风"
      }}&nbsp;
    </span>
    <span class="sm-hidden">{{ weatherData.weather.windpower }}&nbsp;级</span>
  </div>
  <div class="weather" v-else>
    <span>天气数据获取失败</span>
  </div>
</template>

<script setup>
import { getIpLocation, getOpenMeteoWeather } from "@/api";
import { Error } from "@icon-park/vue-next";

// 天气数据
const weatherData = reactive({
  adCode: {
    city: null,
  },
  weather: {
    weather: null,
    temperature: null,
    winddirection: null,
    windpower: null,
  },
});

// 获取天气数据
const getWeatherData = async () => {
  try {
    // 通过 IP 获取城市和坐标
    const location = await getIpLocation();
    weatherData.adCode.city = location.city || location.regionName || "未知地区";

    // 通过 Open-Meteo 获取实时天气
    const meteo = await getOpenMeteoWeather(location.lat, location.lon);
    const current = meteo.current;

    // WMO 代码转中文
    const wmoMap = {
      0: "晴", 1: "晴", 2: "多云", 3: "阴",
      45: "雾", 48: "雾",
      51: "小雨", 53: "中雨", 55: "大雨",
      61: "小雨", 63: "中雨", 65: "大雨",
      71: "小雪", 73: "中雪", 75: "大雪",
      77: "冰粒",
      80: "阵雨", 81: "阵雨", 82: "强阵雨",
      85: "阵雪", 86: "强阵雪",
      95: "雷暴", 96: "雷暴冰雹", 99: "强雷暴冰雹",
    };

    // 风速（m/s）转蒲福风级
    const ms = current.wind_speed_10m;
    const beaufortScale = [0.3, 1.6, 3.4, 5.5, 8.0, 10.8, 13.9, 17.2, 20.8, 24.5, 28.5, 32.7];
    const windpower = beaufortScale.findIndex((v) => ms < v);

    // 风向角度转中文
    const dirs = ["北", "东北", "东", "东南", "南", "西南", "西", "西北"];
    const winddirection = dirs[Math.round(current.wind_direction_10m / 45) % 8];

    weatherData.weather = {
      weather: wmoMap[current.weather_code] ?? "未知",
      temperature: Math.round(current.temperature_2m),
      winddirection,
      windpower: windpower === -1 ? 12 : windpower,
    };
  } catch (error) {
    console.error("天气信息获取失败:", error);
    onError("天气信息获取失败");
  }
};

// 报错信息
const onError = (message) => {
  ElMessage({
    message,
    icon: h(Error, {
      theme: "filled",
      fill: "#efefef",
    }),
  });
  console.error(message);
};

onMounted(() => {
  getWeatherData();
});
</script>
