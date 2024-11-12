<template>
  <div class="q-pa-md contour-info-container">
    <!-- 标题部分 -->
    <div class="text-h6 font-bold">Добавление севооборота в контуре</div>
    <div class="q-mt-md q-gutter-y-xs">
      <div class="info-item row">
        <span class="label col-auto text-subtitle1 font-bold">Сезон:</span>
        <span class="value col text-subtitle1">{{ contourInfo.seasonName }}</span>
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
          :rows="rotationData"
          :columns="columns"
          row-key="cropRotationId"
          hide-bottom
          flat
          bordered
          dense
          class="custom-table full-width"
        >
          <template v-slot:body-cell-edit="props">
            <q-td align="center">
              <q-btn flat round dense icon="edit" color="primary" @click="editCropRotation(props.row.cropRotationId)" />
            </q-td>
          </template>
          <template v-slot:body-cell-delete="props">
            <q-td align="center">
              <q-btn flat round dense icon="delete" color="negative" @click="confirmDelete(props.row.cropRotationId)" />
            </q-td>
          </template>
        </q-table>
        <!-- 添加轮作按钮 -->
        <div class="button-container">
          <q-btn label="Добавить севооборот" @click="goToAddRotationPage" color="primary" class="add-rotation-button" />
        </div>
      </div>
    </q-expansion-item>

    <!-- 删除确认对话框 -->
    <q-dialog v-model="isDeleteDialogOpen" persistent>
      <q-card>
        <q-card-section class="text-h6">
          Вы действительно хотите удалить этот объект?
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Нет" color="primary" @click="isDeleteDialogOpen = false" />
          <q-btn flat label="Да" color="negative" @click="deleteCropRotation" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import { useQuasar } from 'quasar';
import { useRouter } from 'vue-router';
import { userStore } from 'src/usage';

export default {
  name: 'RotationPage',
  setup() {
    const $q = useQuasar();
    const router = useRouter();
    const accessToken = computed(() => userStore.state.access_token);

    // 从路由中获取传递的查询参数
    const contourInfo = ref({
      season: router.currentRoute.value.query.seasonName || 'Unknown Season',
      fieldName: router.currentRoute.value.query.fieldName || 'Unknown Field',
      contourName: router.currentRoute.value.query.contourName || 'Unknown Contour'
    });

    const rotationExpanded = ref(true);
    const rotationData = ref([]);
    const columns = [
      { name: 'culture', label: 'Культура', align: 'left', field: 'culture', sortable: true },
      { name: 'cultivar', label: 'Сорт', align: 'left', field: 'cultivar', sortable: true },
      { name: 'description', label: 'Описание', align: 'left', field: 'description', sortable: true },
      { name: 'startDate', label: 'Дата начала', align: 'left', field: 'startDate', sortable: true },
      { name: 'endDate', label: 'Дата окончания', align: 'left', field: 'endDate', sortable: true },
      { name: 'edit', label: 'Редактировать', align: 'center', field: 'edit', sortable: false },
      { name: 'delete', label: 'Удалить', align: 'center', field: 'delete', sortable: false }
    ];

    const isDeleteDialogOpen = ref(false);
    const selectedCropRotationId = ref(null);

    // 通过 API 获取轮作数据
    const fetchRotationData = async () => {
      const contourId = router.currentRoute.value.query.contourId;
      try {
        const response = await axios.get(`https://34a97d79-460b-4dae-9ff7-1fdaa35a4031.mock.pstmn.io/api/v2/fields-service/contours/${contourId}/crop-rotations`, {
          headers: {
            Authorization: `Bearer ${accessToken.value}`,
            'Content-Type': 'application/json'
          }
        });
        rotationData.value = response.data.map(item => ({
          cropRotationId: item.cropRotationId,
          culture: item.culture || 'Unknown Culture',
          cultivar: item.cultivar || 'Unknown Cultivar',
          description: item.description || 'No Description',
          startDate: item.startDate || 'N/A',
          endDate: item.endDate || 'N/A'
        }));
      } catch (error) {
        console.error('Failed to fetch rotation data:', error);
        $q.notify({
          type: 'negative',
          message: 'Failed to load rotation data. Please try again later.',
          icon: 'error'
        });
      }
    };

    const confirmDelete = (cropRotationId) => {
      selectedCropRotationId.value = cropRotationId;
      isDeleteDialogOpen.value = true;
    };

    const deleteCropRotation = () => {
      console.log(`Deleting crop rotation with ID: ${selectedCropRotationId.value}`);
      isDeleteDialogOpen.value = false;
      // 在这里实现删除逻辑
    };

    onMounted(() => {
      fetchRotationData();
    });

    const goToAddRotationPage = () => {
      router.push('/add_rotation'); // 使用 vue-router 进行跳转
    };

    return {
      contourInfo,
      rotationExpanded,
      rotationData,
      columns,
      isDeleteDialogOpen,
      selectedCropRotationId,
      goToAddRotationPage,
      confirmDelete,
      deleteCropRotation
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

/* 右对齐按钮样式 */
.button-container {
  display: flex;
  justify-content: flex-end;
  margin-top: 16px;
}

.add-rotation-button {
  width: 200px;
  background-color: #2e2e2e;
  color: #fff;
  font-size: 1.1rem;
  border-radius: 8px; /* 圆角 */
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.2); /* 阴影 */
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
