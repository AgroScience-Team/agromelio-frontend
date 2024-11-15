<template>
  <div class="q-pa-md contour-info-container">
    <!-- 信息部分 -->
    <div class="info-section">
      <div class="text-h6 font-bold">Информация о поле</div>
      <div class="q-mt-md q-gutter-y-xs">
        <div class="info-item row">
          <span class="label col-auto text-subtitle1 font-bold">Сезон:</span>
          <span class="value col text-subtitle1">{{ fieldInfo.season }}</span>
        </div>
        <div class="info-item row">
          <span class="label col-auto text-subtitle1 font-bold">Поле:</span>
          <span class="value col text-subtitle1">{{ fieldInfo.fieldName }}</span>
        </div>
      </div>
    </div>

    <!-- 天气数据部分 -->
    <q-expansion-item v-model="weatherExpanded" icon="cloud" dense expand-separator class="q-mt-md weather-section">
      <template v-slot:header>
        <div class="text-h6 font-bold">Данные о погоде</div>
      </template>
      
      <div class="q-pa-md">
        <!-- 显示天气数据 -->
        <div v-if="weatherData" class="weather-data">
          <div v-for="(item, index) in weatherData" :key="index" class="weather-item row weather-item-spacing">
            <span class="label text-subtitle1 font-bold">{{ item.label }}:</span>
            <span class="value text-subtitle1">{{ item.value }}</span>
          </div>
        </div>
        
        <!-- 显示错误信息 -->
        <div v-else-if="weatherError" class="error">
          Ошибка при получении данных о погоде: {{ weatherError.message }}
        </div>

        <!-- 加载状态 -->
        <div v-else class="loading">
          Загрузка данных о погоде...
        </div>
      </div>
    </q-expansion-item>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import { useQuasar } from 'quasar';
import { useRouter } from 'vue-router';
import { userStore } from 'src/usage';

export default {
  name: 'FieldWeatherInfoPage',
  setup() {
    const fieldInfo = ref({
      season: 'Лето 2024',
      fieldName: 'Дальнее поле',
    });

    const weatherExpanded = ref(true);
    const weatherData = ref(null);
    const weatherError = ref(null);
    const accessToken = computed(() => userStore.state.access_token);

    // 请求天气数据的函数
    const fetchMeteoData = async () => {
      const fieldId = '47'; // 假设 fieldId 是已知的，如果动态获取可改为从 route 参数中获取
      const url = `${process.env.VUE_APP_BASE_URL}/api/meteo/${fieldId}`;

      try {
        const response = await axios.get(url, {
          headers: {
            Authorization: `Bearer ${accessToken.value}`,
            'Content-Type': 'application/json',
          },
        });
        
        // 格式化天气数据
        weatherData.value = [
          { label: 'Идентификатор поля', value: response.data.field_id },
          { label: 'Последнее обновление', value: response.data.date_time },
          { label: 'Температура', value: `${response.data.temperature} °C` },
          { label: 'Влажность', value: `${response.data.humidity} %` },
          { label: 'Скорость ветра', value: `${response.data.wind_speed} м/с` },
          { label: 'Осадки', value: `${response.data.precipitation} мм` },
          { label: 'Точка росы', value: `${response.data.dew_point} °C` },
          { label: 'Температура почвы (0 см)', value: `${response.data.soil_temperature_0cm} °C` },
          { label: 'Температура почвы (6 см)', value: `${response.data.soil_temperature_6cm} °C` },
          { label: 'Температура почвы (18 см)', value: `${response.data.soil_temperature_18cm} °C` },
          { label: 'Влажность почвы (0-1 см)', value: `${response.data.soil_moisture_0_to_1cm} %` },
          { label: 'Влажность почвы (1-3 см)', value: `${response.data.soil_moisture_1_to_3cm} %` },
          { label: 'Влажность почвы (3-9 см)', value: `${response.data.soil_moisture_3_to_9cm} %` },
          { label: 'Влажность почвы (9-27 см)', value: `${response.data.soil_moisture_9_to_27cm} %` },
          { label: 'Максимальная температура', value: `${response.data.temperature_max} °C` },
          { label: 'Минимальная температура', value: `${response.data.temperature_min} °C` },
          { label: 'Восход солнца', value: response.data.sunrise },
          { label: 'Закат солнца', value: response.data.sunset },
          { label: 'Суммарные осадки', value: `${response.data.precipitation_sum} мм` },
        ];
      } catch (error) {
        console.error('Error fetching meteo data:', error.response ? error.response.data : error.message);
        weatherError.value = error.response ? error.response.data : { message: error.message };
      }
    };

    // 在组件挂载时获取天气数据
    onMounted(() => {
      fetchMeteoData();
    });

    return {
      fieldInfo,
      weatherExpanded,
      weatherData,
      weatherError,
    };
  }
};
</script>

<style scoped>
/* 主容器样式，控制布局 */
.contour-info-container {
  max-width: 100%;
  display: flex;
  flex-direction: column;
}

/* 信息部分宽度控制 */
.info-section {
  max-width: 60%;
}

/* 天气数据部分宽度控制 */
.weather-section {
  max-width: 80%;
  margin-top: 16px;
}

/* 标题和字体样式 */
.text-h6 {
  font-size: 1.5rem;
  color: #333;
}

.font-bold {
  font-weight: bold;
}

.info-item, .weather-item {
  align-items: center;
  font-size: 1.2rem;
}

/* 标签和内容的对齐与间距调整 */
.label {
  font-weight: bold;
  width: 200px; /* 增加标签宽度以拉开间距 */
}

.value {
  flex-grow: 1;
  margin-left: 20px; /* 设置数据和标签之间的距离 */
}

/* 天气数据项之间的额外间距 */
.weather-item-spacing {
  margin-bottom: 12px;
}

/* 错误消息样式 */
.error {
  color: #ff0000;
  padding: 20px;
  background-color: #ffe5e5;
  border: 1px solid #ff0000;
  border-radius: 8px;
}

/* 加载状态样式 */
.loading {
  color: #555;
  font-size: 16px;
  text-align: center;
}
</style>
