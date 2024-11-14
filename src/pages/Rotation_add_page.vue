<template>
  <div class="q-pa-md rotation-info-container">
    <!-- 页面标题 -->
    <div class="text-h5 font-bold">{{ isEditMode ? 'Редактировать севооборот' : 'Добавление севооборота в контуре' }}</div>

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
      <div class="info-item row">
        <span class="label col-auto text-subtitle1 font-bold">Контур:</span>
        <span class="value col text-subtitle1">{{ contour }}</span>
      </div>
    </div>

    <!-- 输入表单 -->
    <div class="q-mt-md">
      <q-input v-model="formData.culture" label="Название культуры" outlined dense class="q-mb-md"></q-input>
      <q-input v-model="formData.cultivar" label="Название сорта" outlined dense class="q-mb-md"></q-input>
      <q-input v-model="formData.description" label="Описание" outlined dense class="q-mb-md"></q-input>
      <q-input v-model="formData.startDate" label="Дата сева" hint="Формат: DD-MM-YYYY" mask="##-##-####" outlined dense class="q-mb-md"></q-input>
      <q-input v-model="formData.endDate" label="Дата уборки" hint="Формат: DD-MM-YYYY" mask="##-##-####" outlined dense class="q-mb-md"></q-input>
    </div>

    <!-- 文件上传和保存按钮 -->
    <div class="button-section q-mt-md row no-gutters items-center">
      <q-btn label="Загрузить файл" @click="uploadFile" outline color="primary" class="q-mr-md" />
      <input type="file" ref="fileInput" @change="handleFileUpload" style="display: none" />
      
      <q-btn :label="isEditMode ? 'Сохранить изменения' : 'Сохранить'" @click="saveRotation" color="primary" push />
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import axios from 'axios';
import { useQuasar } from 'quasar';
import { userStore } from 'src/usage';

export default {
  name: 'add_rotation',
  setup() {
    const $q = useQuasar();
    const route = useRoute();
    const router = useRouter();
    const accessToken = computed(() => userStore.state.access_token);
    
    const season = ref(route.query.seasonName || '');
    const field = ref(route.query.fieldName || '');
    const contour = ref(route.query.contourName || '');
    const contourId = ref(route.query.contourId);

    const isEditMode = ref(!!route.query.cropRotationId);
    const cropRotationId = ref(route.query.cropRotationId || '');

    const formData = ref({
      culture: route.query.culture || '',
      cultivar: route.query.cultivar || '',
      description: route.query.description || '',
      startDate: route.query.startDate || '',
      endDate: route.query.endDate || ''
    });

    const fileInput = ref(null);

    const uploadFile = () => {
      fileInput.value.click();
    };

    const handleFileUpload = (event) => {
      const file = event.target.files[0];
      if (file) {
        console.log('Файл загружен:', file.name);
      }
    };

    const saveRotation = async () => {
      const apiUrl = `https://34a97d79-460b-4dae-9ff7-1fdaa35a4031.mock.pstmn.io/api/v2/fields-service`;
      const payload = {
        startDate: formData.value.startDate,     // 作物轮作的开始日期
        endDate: formData.value.endDate,         // 作物轮作的结束日期
        description: formData.value.description, // 作物轮作的描述
        culture: formData.value.culture,         // 作物的种类
        cultivar: formData.value.cultivar || ''  // 作物的品种（可选）
      };

      try {
        if (isEditMode.value) {
          // 编辑模式下调用 PUT API 更新作物轮作
          await axios.put(`${apiUrl}/crop-rotation`, payload, {
            params: { cropRotationId: cropRotationId.value },
            headers: {
              Authorization: `Bearer ${accessToken.value}`,
              'Content-Type': 'application/json'
            }
          });
          $q.notify({
            type: 'positive',
            message: 'Изменения успешно сохранены',
            icon: 'check_circle'
          });
        } else {
          // 创建模式下调用 POST API
          await axios.post(`${apiUrl}/contours/${contourId.value}/crop-rotation`, payload, {
            headers: {
              Authorization: `Bearer ${accessToken.value}`,
              'Content-Type': 'application/json'
            }
          });
          $q.notify({
            type: 'positive',
            message: 'Новый севооборот успешно добавлен',
            icon: 'check_circle'
          });
        }
        router.push({ name: 'RotationPage' }); // 保存后返回到 RotationPage 页面
      } catch (error) {
        console.error('Ошибка при сохранении данных:', error);
        $q.notify({
          type: 'negative',
          message: 'Ошибка при сохранении данных. Попробуйте еще раз.',
          icon: 'error'
        });
      }
    };

    return {
      season,
      field,
      contour,
      formData,
      isEditMode,
      fileInput,
      uploadFile,
      handleFileUpload,
      saveRotation
    };
  }
};
</script>

<style scoped>
.rotation-info-container {
  max-width: 700px;
  margin-left: 0;
}

.text-h5 {
  font-size: 1.5rem;
  color: #333;
}

.font-bold {
  font-weight: bold;
}

.info-section {
  margin-top: 16px;
}

.info-item {
  align-items: center;
  font-size: 1.2rem;
}

.label {
  font-weight: bold;
  width: 120px;
}

.value {
  flex-grow: 1;
}

.button-section {
  display: flex;
  gap: 16px;
}
</style>
