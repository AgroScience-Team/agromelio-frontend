<template>
  <div class="q-pa-md contour-info-container">
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
              <q-btn
                flat
                round
                dense
                icon="edit"
                color="primary"
                @click="editCropRotation(props.row)"
              />
            </q-td>
          </template>
          <template v-slot:body-cell-delete="props">
            <q-td align="center">
              <q-btn
                flat
                round
                dense
                icon="delete"
                color="negative"
                @click="confirmDelete(props.row.cropRotationId, 'cropRotation')"
              />
            </q-td>
          </template>
        </q-table>
        <div class="button-container">
          <q-btn label="Добавить севооборот" @click="goToAddRotationPage" color="primary" class="button-common" />
        </div>
      </div>
    </q-expansion-item>

    <q-expansion-item v-model="soilExpanded" icon="nature" dense expand-separator class="q-mt-md full-width">
      <template v-slot:header>
        <div class="text-h6 font-bold">Данные о почве</div>
      </template>

      <div class="q-pa-md full-width">
        <div class="table-scroll-container">
          <q-table
            flat
            bordered
            dense
            class="custom-table"
            :rows="[...soilData]"
            :columns="soilColumns"
            row-key="parameter"
            :rows-per-page-options="[0]"
            style="table-layout: fixed;"
          >
            <template v-slot:body="props">
              <q-tr v-if="props.row.parameter !== 'edit' && props.row.parameter !== 'delete'">
                <q-td>{{ props.row.parameter }}</q-td>
                <q-td
                  v-for="(column, index) in soilColumns.slice(1)"
                  :key="index"
                  align="center"
                >
                  {{ props.row[column.name] || '—' }}
                </q-td>
              </q-tr>

              <q-tr v-else-if="props.row.parameter === 'edit'">
                <q-td>Редактировать</q-td>
                <q-td
                  v-for="(column, index) in soilColumns.slice(1)"
                  :key="index"
                  align="center"
                >
                  <q-icon
                    name="edit"
                    color="primary"
                    size="md"
                    class="cursor-pointer"
                    @click="editSoilInfo(getSoilInfoById(column.name))"
                  />
                </q-td>
              </q-tr>

              <q-tr v-else-if="props.row.parameter === 'delete'">
                <q-td>Удалить</q-td>
                <q-td
                  v-for="(column, index) in soilColumns.slice(1)"
                  :key="index"
                  align="center"
                >
                  <q-icon
                    name="delete"
                    color="negative"
                    size="md"
                    class="cursor-pointer"
                    @click="confirmDelete(column.name, 'soilComposition')"
                  />
                </q-td>
              </q-tr>
            </template>
          </q-table>
        </div>
        <div class="button-container">
          <q-btn
            label="Добавить данные о составе почвы"
            @click="goToAddSoilPage"
            color="primary"
            class="button-common"
          />
        </div>
      </div>
    </q-expansion-item>

    <q-dialog v-model="isDeleteDialogOpen" persistent>
      <q-card>
        <q-card-section class="text-h6">
          Вы действительно хотите удалить этот объект?
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Нет" color="primary" @click="isDeleteDialogOpen = false" />
          <q-btn flat label="Да" color="negative" @click="deleteItem()" />
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

    const contourInfo = computed(() => ({
      seasonName: router.currentRoute.value.query.seasonName || 'Unknown Season',
      fieldName: router.currentRoute.value.query.fieldName || 'Unknown Field',
      contourName: router.currentRoute.value.query.contourName || 'Unknown Contour',
      contourId: router.currentRoute.value.query.contourId || 'Unknown Contour ID'
    }));

    const rotationExpanded = ref(true);
    const rotationData = ref([]);
    const soilExpanded = ref(false);
    const deleteType = ref(null);

    const columns = [
      { name: 'culture', label: 'Культура', align: 'left', field: 'culture', sortable: true },
      { name: 'cultivar', label: 'Сорт', align: 'left', field: 'cultivar', sortable: true },
      { name: 'description', label: 'Описание', align: 'left', field: 'description', sortable: true },
      { name: 'startDate', label: 'Дата начала', align: 'left', field: 'startDate', sortable: true },
      { name: 'endDate', label: 'Дата окончания', align: 'left', field: 'endDate', sortable: true },
      { name: 'edit', label: 'Редактировать', align: 'center', field: 'edit', sortable: false },
      { name: 'delete', label: 'Удалить', align: 'center', field: 'delete', sortable: false }
    ];

    const soilParameters = [
      { key: 'ph', name: 'pH' },
      { key: 'organicMatter', name: 'Organic Matter' },
      { key: 'mobileP', name: 'Mobile P' },
      { key: 'mobileK', name: 'Mobile K' },
      { key: 'mobileS', name: 'Mobile S' },
      { key: 'nitrateN', name: 'Nitrate N' },
      { key: 'ammoniumN', name: 'Ammonium N' },
      { key: 'hydrolyticAcidity', name: 'Hydrolytic Acidity' },
      { key: 'caExchange', name: 'Ca Exchange' },
      { key: 'mgExchange', name: 'Mg Exchange' },
      { key: 'b', name: 'B' },
      { key: 'co', name: 'Co' },
      { key: 'mn', name: 'Mn' },
      { key: 'zn', name: 'Zn' }
    ];

    const soilColumns = ref([
      { name: 'parameter', label: 'Параметр', align: 'left', field: 'parameter', sortable: false },
      { name: 'edit', label: 'Редактировать', align: 'center', field: 'edit', sortable: false },
      { name: 'delete', label: 'Удалить', align: 'center', field: 'delete', sortable: false },
    ]);
    const soilData = ref([]);

    const isDeleteDialogOpen = ref(false);
    const selectedCropRotationId = ref(null);
    const selectedSoilInfoId = ref(null);

    // fake data
    const fakeRotationData = [
      {
        id: "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        culture: "Пшеница",
        cultivar: "Сорт A",
        startDate: "2024-03-01",
        endDate: "2024-06-15",
        description: "Высокое качество зерна, урожай 2024"
      },
      {
        id: "3fa85f64-5717-4562-b3fc-2c963f66afa7",
        culture: "Кукуруза",
        cultivar: "Сорт B",
        startDate: "2024-07-01",
        endDate: "2024-10-10",
        description: "Кукуруза для корма"
      },
      {
        id: "3fa85f64-5717-4562-b3fc-2c963f66afa8",
        culture: "Овёс",
        cultivar: "Сорт C",
        startDate: "2024-10-20",
        endDate: "2025-01-15",
        description: "Овёс для животноводства"
      }
    ];

    const fakeSoilData = [
      {
        id: "1a2b3c4d-5678-90ab-cdef-1234567890ab",
        ph: "6.5",
        sampleDate: "2024-11-20",
        organicMatter: "3.2%",
        mobileP: "15 ppm",
        mobileK: "120 ppm",
        mobileS: "10 ppm",
        nitrateN: "5 ppm",
        ammoniumN: "3 ppm",
        hydrolyticAcidity: "4.8",
        caExchange: "20 meq/100g",
        mgExchange: "5 meq/100g",
        b: "0.2 ppm",
        co: "0.1 ppm",
        mn: "50 ppm",
        zn: "2 ppm"
      },
      {
        id: "2b3c4d5e-6789-01ab-cdef-2345678901bc",
        ph: "6.8",
        sampleDate: "2024-11-22",
        organicMatter: "3.0%",
        mobileP: "12 ppm",
        mobileK: "115 ppm",
        mobileS: "8 ppm",
        nitrateN: "6 ppm",
        ammoniumN: "4 ppm",
        hydrolyticAcidity: "4.7",
        caExchange: "18 meq/100g",
        mgExchange: "6 meq/100g",
        b: "0.25 ppm",
        co: "0.15 ppm",
        mn: "45 ppm",
        zn: "1.8 ppm"
      },
      {
        id: "3c4d5e6f-7890-12ab-cdef-3456789012cd",
        ph: "7.0",
        sampleDate: "2024-11-24",
        organicMatter: "2.8%",
        mobileP: "10 ppm",
        mobileK: "100 ppm",
        mobileS: "12 ppm",
        nitrateN: "4 ppm",
        ammoniumN: "2 ppm",
        hydrolyticAcidity: "4.5",
        caExchange: "22 meq/100g",
        mgExchange: "7 meq/100g",
        b: "0.3 ppm",
        co: "0.2 ppm",
        mn: "40 ppm",
        zn: "1.5 ppm"
      },
      {
        id: "3c3d5e6f-7890-12ab-cdef-3456789012cd",
        ph: "7.0",
        sampleDate: "2024-11-26",
        organicMatter: "2.8%",
        mobileP: "10 ppm",
        mobileK: "100 ppm",
        mobileS: "12 ppm",
        nitrateN: "4 ppm",
        ammoniumN: "2 ppm",
        hydrolyticAcidity: "4.5",
        caExchange: "22 meq/100g",
        mgExchange: "7 meq/100g",
        b: "0.3 ppm",
        co: "0.2 ppm",
        mn: "40 ppm",
        zn: "1.5 ppm"
      },
      {
        id: "3c4d5e6f-7890-00ab-cdef-3456789012cd",
        ph: "7.0",
        sampleDate: "2024-11-27",
        organicMatter: "2.8%",
        mobileP: "10 ppm",
        mobileK: "100 ppm",
        mobileS: "12 ppm",
        nitrateN: "4 ppm",
        ammoniumN: "2 ppm",
        hydrolyticAcidity: "4.5",
        caExchange: "22 meq/100g",
        mgExchange: "7 meq/100g",
        b: "0.3 ppm",
        co: "0.2 ppm",
        mn: "40 ppm",
        zn: "1.5 ppm"
      },
      {
        id: "3c4d5e6f-7890-12ab-cdef-3412389012cd",
        ph: "7.0",
        sampleDate: "2024-11-29",
        organicMatter: "2.8%",
        mobileP: "10 ppm",
        mobileK: "100 ppm",
        mobileS: "12 ppm",
        nitrateN: "4 ppm",
        ammoniumN: "2 ppm",
        hydrolyticAcidity: "4.5",
        caExchange: "22 meq/100g",
        mgExchange: "7 meq/100g",
        b: "0.3 ppm",
        co: "0.2 ppm",
        mn: "40 ppm",
        zn: "1.5 ppm"
      },
      {
        id: "3c4d5e6f-7890-12ab-cdef-3412119012cd",
        ph: "7.0",
        sampleDate: "2024-12-29",
        organicMatter: "2.8%",
        mobileP: "10 ppm",
        mobileK: "100 ppm",
        mobileS: "12 ppm",
        nitrateN: "4 ppm",
        ammoniumN: "2 ppm",
        hydrolyticAcidity: "4.5",
        caExchange: "22 meq/100g",
        mgExchange: "7 meq/100g",
        b: "0.3 ppm",
        co: "0.2 ppm",
        mn: "40 ppm",
        zn: "1.5 ppm"
      }
    ];


    const fetchRotationData = async () => {
      const contourId = contourInfo.value.contourId;
      if (!contourId) {
        console.error('Invalid contour ID');
        return;
      }
      try {
        const response = await axios.get(`${process.env.VUE_APP_BASE_URL}/api/fields-service/contours/${contourId}/crop-rotations`, {
          headers: {
            Authorization: `Bearer ${accessToken.value}`,
            'Content-Type': 'application/json'
          }
        });
        rotationData.value = response.data.map(item => ({
          cropRotationId: item.id,
          culture: item.culture,
          cultivar: item.cultivar || 'N/A',
          description: item.description,
          startDate: item.startDate,
          endDate: item.endDate
        }));
      } catch (error) {
        console.error('Failed to fetch rotation data:', error);
        $q.notify({
          type: 'negative',
          message: 'Failed to load rotation data. Please try again later.',
          icon: 'error'
        });

        // 请求失败时使用假数据
        rotationData.value = fakeRotationData.map(item => ({
          cropRotationId: item.id,
          culture: item.culture,
          cultivar: item.cultivar,
          description: item.description,
          startDate: item.startDate,
          endDate: item.endDate
        }));
      }
    };

    const fetchSoilData = async () => {
      const contourId = contourInfo.value.contourId;
    
      if (!contourId || contourId === 'Unknown Contour ID') {
        console.error('Invalid contour ID');
        return;
      }
    
      try {
        const response = await axios.get(
          `${process.env.VUE_APP_BASE_URL}/api/fields-service/contours/${contourId}/soil-compositions`,
          {
            headers: {
              Authorization: `Bearer ${accessToken.value}`,
              'Content-Type': 'application/json',
            },
          }
        );
    
        let data = response.data;
    
        if (data.length > 0) {
          // 按日期排序
          const sortedData = [...data].sort((a, b) => 
            new Date(a.sampleDate) - new Date(b.sampleDate)
          );
    
          // 生成列配置
          const dataColumns = sortedData.map(item => ({
            name: item.id,
            label: item.sampleDate,
            align: 'center',
            field: item.id,
            sortable: true
          }));
    
          // 设置列
          soilColumns.value = [
            { name: 'parameter', label: 'Параметр', align: 'left', field: 'parameter', sortable: false },
            ...dataColumns
          ];
    
          // 生成行数据
          soilData.value = soilParameters.map(param => {
            const row = { parameter: param.name };
            sortedData.forEach(item => {
              row[item.id] = item[param.key] || '—';
            });
            return row;
          });
    
          // 添加编辑和删除行
          soilData.value.push({ parameter: 'edit' });
          soilData.value.push({ parameter: 'delete' });
        }
      } catch (error) {
        console.error('Failed to fetch soil data:', error);
        $q.notify({
          type: 'negative',
          message: 'Failed to load soil data. Please try again later.',
          icon: 'error',
        });
    
        // 使用假数据
        const sortedData = [...fakeSoilData].sort((a, b) => 
          new Date(a.sampleDate) - new Date(b.sampleDate)
        );
    
        // 使用ID生成列
        const dataColumns = sortedData.map(item => ({
          name: item.id,
          label: item.sampleDate,
          align: 'center',
          field: item.id,
          sortable: true
        }));
    
        // 设置列
        soilColumns.value = [
          { name: 'parameter', label: 'Параметр', align: 'left', field: 'parameter', sortable: false },
          ...dataColumns
        ];
    
        // 生成行数据
        soilData.value = soilParameters.map(param => {
          const row = { parameter: param.name };
          sortedData.forEach(item => {
            row[item.id] = item[param.key] || '—';
          });
          return row;
        });
    
        // 添加编辑和删除行
        soilData.value.push({ parameter: 'edit' });
        soilData.value.push({ parameter: 'delete' });
      }
    };

    const getSoilInfoById = (id) => {
      const soilInfo = fakeSoilData.find(item => item.id === id);
      if (!soilInfo) {
        console.error(`No soil data found for id: ${id}`);
        return null;
      }
      return {
        id: soilInfo.id,
        ph: soilInfo.ph,
        sampleDate: soilInfo.sampleDate,
        organicMatter: soilInfo.organicMatter,
        mobileP: soilInfo.mobileP,
        mobileK: soilInfo.mobileK,
        mobileS: soilInfo.mobileS,
        nitrateN: soilInfo.nitrateN,
        ammoniumN: soilInfo.ammoniumN,
        hydrolyticAcidity: soilInfo.hydrolyticAcidity,
        caExchange: soilInfo.caExchange,
        mgExchange: soilInfo.mgExchange,
        b: soilInfo.b,
        co: soilInfo.co,
        mn: soilInfo.mn,
        zn: soilInfo.zn
      };
    };

    const confirmDelete = (itemId, itemType = 'cropRotation') => {
      // 根据类型设置对应的选中 ID 和类型
      deleteType.value = itemType; // 保存当前删除类型
      if (itemType === 'cropRotation') {
        selectedCropRotationId.value = itemId;
      } else if (itemType === 'soilComposition') {
        selectedSoilInfoId.value = itemId;
      }

      // 打开确认删除对话框
      isDeleteDialogOpen.value = true;
    };

    const deleteItem = async () => {
      try {
        if (deleteType.value === 'cropRotation') {
          // 删除轮作信息
          const cropRotationId = selectedCropRotationId.value;
          const response = await axios.delete(`${process.env.VUE_APP_BASE_URL}/api/fields-service/crop-rotation`, {
            params: { cropRotationId },
            headers: {
              Authorization: `Bearer ${accessToken.value}`,
              'Content-Type': 'application/json',
            },
          });

          if (response.status === 200) {
            $q.notify({
              type: 'positive',
              message: 'Crop rotation deleted successfully!',
              icon: 'check_circle',
            });
            await fetchRotationData(); // 重新加载轮作数据
          } else {
            throw new Error('Failed to delete crop rotation.');
          }
        } else if (deleteType.value === 'soilComposition') {
          // 删除土壤信息
          const soilCompositionId = selectedSoilInfoId.value;
          const response = await axios.delete(`${process.env.VUE_APP_BASE_URL}/api/fields-service/soil-composition`, {
            params: { soilCompositionId },
            headers: {
              Authorization: `Bearer ${accessToken.value}`,
              'Content-Type': 'application/json',
            },
          });

          if (response.status === 200) {
            $q.notify({
              type: 'positive',
              message: 'Информация о почве успешно удалена!',
              icon: 'check_circle',
            });
            await fetchSoilData(); // 重新加载土壤数据
          } else {
            throw new Error('Failed to delete soil composition.');
          }
        }
      } catch (error) {
        console.error('Delete operation failed:', error);
        $q.notify({
          type: 'negative',
          message: 'Ошибка при удалении данных. Попробуйте позже.',
          icon: 'error',
        });
      } finally {
        isDeleteDialogOpen.value = false;
        selectedCropRotationId.value = null;
        selectedSoilInfoId.value = null;
        deleteType.value = null; // 重置删除类型
      }
    };

    const editSoilInfo = (soilInfo) => {
      if (!soilInfo) {
        console.error("Invalid soil information object");
        return;
      }

      // 跳转到编辑页面，并传递所需的查询参数
      router.push({
        name: 'add_soil', // 假设编辑土地信息的路由名称是 "add_soil"
        query: {
          contourId: contourInfo.value.contourId, // 当前轮廓 ID
          seasonName: contourInfo.value.seasonName, // 当前季节名称
          fieldName: contourInfo.value.fieldName, // 当前字段名称
          contourName: contourInfo.value.contourName, // 当前轮廓名称
          soilInfoId: soilInfo.id, // 土壤信息的唯一 ID
          ph: soilInfo.ph,
          sampleDate: soilInfo.sampleDate,
          organicMatter: soilInfo.organicMatter,
          mobileP: soilInfo.mobileP,
          mobileK: soilInfo.mobileK,
          mobileS: soilInfo.mobileS,
          nitrateN: soilInfo.nitrateN,
          ammoniumN: soilInfo.ammoniumN,
          hydrolyticAcidity: soilInfo.hydrolyticAcidity,
          caExchange: soilInfo.caExchange,
          mgExchange: soilInfo.mgExchange,
          b: soilInfo.b,
          co: soilInfo.co,
          mn: soilInfo.mn,
          zn: soilInfo.zn,
        },
      });
    };

    const editCropRotation = (cropRotation) => {
      router.push({
        name: 'add_rotation',
        query: {
          contourId: contourInfo.value.contourId,
          seasonName: contourInfo.value.seasonName,
          fieldName: contourInfo.value.fieldName,
          contourName: contourInfo.value.contourName,
          cropRotationId: cropRotation.cropRotationId,
          culture: cropRotation.culture,
          cultivar: cropRotation.cultivar,
          description: cropRotation.description,
          startDate: cropRotation.startDate,
          endDate: cropRotation.endDate
        }
      });
    };

    onMounted(() => {
      fetchRotationData();
      fetchSoilData();
    });

    const goToAddRotationPage = () => {
      router.push({
        name: 'add_rotation',
        query: {
          contourId: contourInfo.value.contourId,
          seasonName: contourInfo.value.seasonName,
          fieldName: contourInfo.value.fieldName,
          contourName: contourInfo.value.contourName
        }
      });
    };

    const goToAddSoilPage = () => {
      router.push({
        name: 'add_soil', 
        query: {
          contourId: contourInfo.value.contourId, 
          seasonName: contourInfo.value.seasonName, 
          fieldName: contourInfo.value.fieldName, 
          contourName: contourInfo.value.contourName
        }
      });
    };


    return {
      contourInfo,
      rotationExpanded,
      rotationData,
      columns,
      isDeleteDialogOpen,
      selectedCropRotationId,
      selectedSoilInfoId,
      soilData,
      soilColumns,
      soilExpanded,
      fetchRotationData,
      fetchSoilData,
      confirmDelete,
      editCropRotation,
      goToAddRotationPage,
      goToAddSoilPage,
      editSoilInfo,
      deleteItem,
      getSoilInfoById
    };
  }
};
</script>

<style scoped>
/* 页面整体滚动 */
html, body {
  height: 100%;
  overflow-y: auto; /* 启用垂直滚动 */
  margin: 0;
}

/* 容器基础样式 */
.contour-info-container {
  max-width: 100%;
  margin-left: 0;
}

.full-width {
  width: 100%;
}

/* 标题样式 */
.text-h6 {
  font-size: 1.25rem;
  color: #333;
  font-weight: bold;
}

/* 信息项样式 */
.info-item {
  align-items: center;
  font-size: 1.2rem;
}

.label {
  font-weight: bold;
  width: 100px;
}

.value {
  flex-grow: 1;
}

/* 按钮容器 */
.button-container {
  display: flex;
  justify-content: flex-end;
  margin-top: 16px;
}

/* 统一按钮样式 */
.button-common {
  width: 200px; /* 确保宽度一致 */
  background-color: #2e2e2e; /* 使用深色背景 */
  color: #fff; /* 白色文字 */
  font-size: 1.1rem; /* 字体大小一致 */
  border-radius: 8px; /* 按钮圆角 */
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.2); /* 添加阴影 */
  text-transform: none; /* 禁止文字大写 */
}


.table-scroll-container {
  overflow-x: auto; /* 启用水平滚动 */
  white-space: nowrap; /* 防止换行 */
  border: 1px solid #ccc;
  padding: 8px;
  margin-top: 16px;
}

.custom-table .q-table__cell {
  max-width: 5000px !important; /* 每列最大宽度 */
  min-width: 5000px !important;
  text-overflow: ellipsis; /* 超出部分显示省略号 */
  overflow: hidden; /* 超出部分隐藏 */
  white-space: nowrap; /* 不允许换行 */
}

.custom-table .q-table__title {
  max-width: 5000px; /* 每列最大宽度 */
  min-width: 5000px;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}

.q-table__title {
  font-weight: bold;
  color: #333;
}
</style>