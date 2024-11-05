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
          <div v-for="(item, index) in weatherData" :key="index" class="weather-item row weather-item-spacing">
            <span class="label text-subtitle1 font-bold">{{ item.label }}:</span>
            <span class="value text-subtitle1">{{ item.value }}</span>
          </div>
        </div>
      </q-expansion-item>
    </div>
  </template>
  
  <script>
  import { ref } from 'vue';
  
  export default {
    name: 'FieldInfoPage',
    setup() {
      const fieldInfo = ref({
        season: 'Лето 2024',
        fieldName: 'Дальнее поле',
      });
  
      const weatherExpanded = ref(true);
      const weatherData = ref([
        { label: 'Идентификатор поля', value: '47' },
        { label: 'Последнее обновление', value: '2024-10-26T18:00:00' },
        { label: 'Температура', value: '8.1 °C' },
        { label: 'Влажность', value: '84 %' },
        { label: 'Скорость ветра', value: '2 м/с' },
        { label: 'Осадки', value: '0 мм' },
        { label: 'Точка росы', value: '5.5 °C' }
      ]);
  
      return {
        fieldInfo,
        weatherExpanded,
        weatherData,
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
  </style>
  