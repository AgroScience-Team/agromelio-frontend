<template>
    <div class="q-pa-md rotation-info-container">
      <!-- 页面标题 -->
      <div class="text-h5 font-bold">Добавление севооборота в контуре</div>
  
      <!-- 基本信息部分 -->
      <div class="q-mt-md q-gutter-y-xs info-section">
        <div class="info-item row">
          <span class="label col-auto text-subtitle1 font-bold">Сезон:</span>
          <span class="value col text-subtitle1">{{ season }}</span>
        </div>
        <div class="info-item row">
          <span class="label col-auto text-subtitle1 font-bold">Поле:</span>
          <span class="value col text-subtitle1">{{ field }}</span>
        </div>
      </div>
  
      <!-- 输入表单 -->
      <div class="q-mt-md">
        <q-input v-model="formData.cultureName" label="Название культуры" outlined dense class="q-mb-md"></q-input>
        <q-input v-model="formData.sortName" label="Название сорта" outlined dense class="q-mb-md"></q-input>
        <q-input v-model="formData.description" label="Описание" outlined dense class="q-mb-md"></q-input>
        <q-input v-model="formData.sowingDate" label="Дата сева" hint="Формат: DD-MM-YYYY" mask="##-##-####" outlined dense class="q-mb-md"></q-input>
        <q-input v-model="formData.harvestDate" label="Дата уборки" hint="Формат: DD-MM-YYYY" mask="##-##-####" outlined dense class="q-mb-md"></q-input>
      </div>
  
      <!-- 文件上传和保存按钮 -->
      <div class="button-section q-mt-md">
        <q-btn label="Загрузить файл" @click="uploadFile" outline color="primary" class="upload-button" />
        <input type="file" ref="fileInput" @change="handleFileUpload" style="display: none" />
      </div>
      <q-btn label="Сохранить" @click="saveRotation" color="primary" class="save-button q-mt-md" />
    </div>
  </template>
  
  <script>
  import { ref } from 'vue';
  
  export default {
    name: 'AddRotationPage',
    setup() {
      const season = 'Лето 2024';
      const field = 'Дальнее поле';
      const formData = ref({
        cultureName: '',
        sortName: '',
        description: '',
        sowingDate: '',
        harvestDate: ''
      });
  
      const fileInput = ref(null);
  
      const uploadFile = () => {
        fileInput.value.click();
      };
  
      const handleFileUpload = (event) => {
        const file = event.target.files[0];
        if (file) {
          // 文件上传处理逻辑
          console.log('Файл загружен:', file.name);
        }
      };
  
      const saveRotation = () => {
        // 保存表单数据的逻辑
        console.log('Form data:', formData.value);
      };
  
      return {
        season,
        field,
        formData,
        fileInput,
        uploadFile,
        handleFileUpload,
        saveRotation
      };
    }
  };
  </script>
  
  <style scoped>
  /* 主要容器样式 */
  .rotation-info-container {
    max-width: 700px;
    margin-left: 0;
  }
  
  /* 标题样式 */
  .text-h5 {
    font-size: 1.5rem;
    color: #333;
  }
  
  /* 信息部分样式 */
  .info-section {
    margin-top: 16px;
  }
  
  .font-bold {
    font-weight: bold;
  }
  
  .info-item {
    align-items: center;
    font-size: 1.2rem;
  }
  
  /* 标签和内容的对齐 */
  .label {
    font-weight: bold;
    width: 120px;
  }
  
  .value {
    flex-grow: 1;
  }
  
  /* 上传按钮和保存按钮样式 */
  .upload-button {
    font-size: 1rem;
    margin-right: 16px;
  }
  
  .save-button {
    width: 200px;
    background-color: #2e2e2e;
    color: #fff;
    font-size: 1.1rem;
  }
  </style>
  