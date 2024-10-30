<template>
    <div class="q-pa-md contour-info-container">
      <!-- 标题部分 -->
      <div class="text-h6 font-bold">Информация о контуре</div>
      <div class="q-mt-md q-gutter-y-xs">
        <div class="info-item row">
          <span class="label col-auto text-subtitle1 font-bold">Сезон:</span>
          <span class="value col text-subtitle1">{{ contourInfo.season }}</span>
        </div>
        <div class="info-item row">
          <span class="label col-auto text-subtitle1 font-bold">Поле:</span>
          <span class="value col text-subtitle1">{{ contourInfo.fieldName }}</span>
        </div>
        <div class="info-item row">
          <span class="label col-auto text-subtitle1 font-bold">Контур:</span>
          <span class="value col text-subtitle1">{{ contourInfo.contourName }}</span>
        </div>
      </div>
  
      <!-- 轮作信息部分 -->
      <q-expansion-item v-model="rotationExpanded" icon="eco" dense expand-separator class="q-mt-md full-width">
        <template v-slot:header>
          <div class="text-h6 font-bold">Севооборот</div>
        </template>
        
        <div class="q-pa-md full-width">
          <q-table
            :rows="displayedRotationData"
            :columns="columns"
            row-key="id"
            hide-bottom
            flat
            bordered
            dense
            class="custom-table full-width"
          />
          <!-- 添加轮作按钮，使用 router.push 来保持与 add-field 页面的逻辑一致 -->
          <q-btn label="Добавить севооборот" @click="goToAddRotationPage" color="primary" class="add-rotation-button q-mt-md" />
        </div>
      </q-expansion-item>
    </div>
  </template>
  
  <script>
  import { ref, onMounted, computed } from 'vue';
  import { useRouter } from 'vue-router';
  import axios from 'axios';
  
  export default {
    name: 'ContourInfoPage',
    setup() {
      const router = useRouter();
      const contourInfo = ref({
        season: 'Лето 2024',
        fieldName: 'Дальнее поле',
        contourName: 'Название контура'
      });
  
      const rotationExpanded = ref(true);
      const rotationData = ref([]);
      const columns = [
        { name: 'culture', label: 'Культура', align: 'left', field: row => row.culture, sortable: true },
        { name: 'sort', label: 'Сорт', align: 'left', field: row => row.sort, sortable: true },
        { name: 'description', label: 'Описание', align: 'left', field: row => row.description, sortable: true },
        { name: 'startDate', label: 'Дата начала', align: 'left', field: row => row.startDate, sortable: true },
        { name: 'endDate', label: 'Дата окончания', align: 'left', field: row => row.endDate, sortable: true },
        { name: 'edit', label: 'Редактировать', align: 'center', field: 'edit', sortable: false },
        { name: 'delete', label: 'Удалить', align: 'center', field: 'delete', sortable: false }
      ];
  
      // 模拟从后端获取数据
      onMounted(() => {
        axios.get('/api/rotation-data') // 假设的API请求
          .then(response => {
            rotationData.value = response.data;
          })
          .catch(error => {
            console.error('Failed to fetch rotation data:', error);
          });
      });
  
      // 计算显示的数据行，如果没有数据则显示提示信息
      const displayedRotationData = computed(() => {
        return rotationData.value.length > 0 ? rotationData.value : [{ culture: 'Нет информации о культурах', sort: '', description: '', startDate: '', endDate: '', edit: '', delete: '' }];
      });
  
      const goToAddRotationPage = () => {
        router.push('/add_rotation'); // 使用 vue-router 进行跳转
      };
  
      return {
        contourInfo,
        rotationExpanded,
        rotationData,
        displayedRotationData,
        columns,
        goToAddRotationPage
      };
    }
  };
  </script>
  
  <style scoped>
  /* 主要容器样式，控制左对齐 */
  .contour-info-container {
    max-width: 100%; /* 设置为 100%，以确保内容能够充满页面 */
    margin-left: 0;
  }
  
  .full-width {
    width: 100%; /* 确保 q-expansion-item 和 q-table 的宽度为 100% */
  }
  
  /* 标题和标签的样式 */
  .text-h6 {
    font-size: 1.25rem;
    color: #333;
  }
  
  .font-bold {
    font-weight: bold;
  }
  
  .info-item {
    align-items: center;
    font-size: 1.2rem; /* 增大字体 */
  }
  
  /* 标签和内容的对齐 */
  .label {
    font-weight: bold;
    width: 100px;
  }
  
  .value {
    flex-grow: 1;
  }
  
  /* 添加轮作按钮样式 */
  .add-rotation-button {
    width: 200px;
    background-color: #2e2e2e;
    color: #fff;
    font-size: 1.1rem;
    margin-top: 16px;
  }
  
  /* 表格样式 */
  .custom-table .q-table__title,
  .custom-table .q-table__cell {
    font-size: 1.1rem; /* 增大表格字体 */
    padding: 8px;
  }
  
  .q-table__title {
    font-weight: bold;
    color: #333;
  }
  </style>
  