<template>
  <q-page padding>
    <div class="q-pa-md q-gutter-md">
      <!-- 下拉选择季节 -->
      <q-card>
        <q-card-section>
          <div class="text-h6">Севооборот</div>
        </q-card-section>
        <q-card-section>
          <q-select
            v-model="selectedSeason"
            label="Выбор сезона"
            :options="seasons"
            dense
            outlined
            @update:model-value="onSeasonChange"
          />
        </q-card-section>
      </q-card>

      <!-- 表格部分 -->
      <q-card class="q-mt-md">
        <q-card-section>
          <div class="text-h6">Информация о культурах</div>
        </q-card-section>
        <q-card-section>
          <q-table
            flat
            bordered
            :rows="fieldsData"
            :columns="fieldsColumns"
            row-key="contour_id"
            v-model:pagination="pagination"
          >
            <!-- 自定义单元格渲染 -->
            <template v-slot:body-cell-field_name="props">
              <q-td :props="props">
                <div>{{ props.row.field_name }}</div>
              </q-td>
            </template>

            <template v-slot:body-cell-contours="props">
              <q-td :props="props">
                <div class="contour-item">
                  {{ props.row.contour_name }} ({{ props.row.squareArea }} га)
                  <!-- 修改按钮 -->
                  <q-btn
                    flat
                    icon="edit"
                    color="primary"
                    @click="navigateToEditPage(props.row.contour_id)"
                    class="edit-button"
                  />
                </div>
              </q-td>
            </template>

            <template v-slot:body-cell-crop_rotations="props">
              <q-td :props="props">
                <div class="crop-rotations">
                  <div
                    v-for="crop in props.row.crop_rotations"
                    :key="crop.cropRotationId"
                    class="crop-rotation"
                  >
                    <span>{{ crop.culture }} ({{ crop.sort }})</span>
                    <div class="timeline">
                      <span class="timeline-dot"></span>
                      <span class="timeline-line"></span>
                      <span class="timeline-date">{{ crop.start_date }}</span> - <span class="timeline-date">{{ crop.end_date }}</span>
                    </div>
                  </div>
                </div>
              </q-td>
            </template>
          </q-table>
        </q-card-section>
      </q-card>
    </div>
  </q-page>
</template>

<script>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import { useQuasar } from 'quasar';
import { useRouter } from 'vue-router';
import { userStore } from 'src/usage';

export default {
  name: 'SeasonPage',
  setup() {
    const router = useRouter();
    const $q = useQuasar();

    const selectedSeason = ref('');
    const selectedSeasonName = ref('');
    const seasons = ref([]); // 季节列表初始化为空
    const fieldsData = ref([]); // 字段数据初始化为空
    const accessToken = computed(() => userStore.state.access_token);
    const pagination = ref({ rowsPerPage: 10 });

    // 列配置
    const fieldsColumns = [
      { name: 'field_name', label: 'поля', align: 'center', field: 'field_name', style: 'width: 20%' },
      { name: 'contours', label: 'Контуры', align: 'center', field: 'contours', style: 'width: 30%' },
      { name: 'crop_rotations', label: 'Посевы', align: 'center', field: 'crop_rotations', style: 'width: 50%' }
    ];

    // 获取季节列表
    onMounted(async () => {
      try {
        const response = await axios.get(`https://34a97d79-460b-4dae-9ff7-1fdaa35a4031.mock.pstmn.io/api/v2/fields-service/seasons`, {
          headers: {
            Authorization: `Bearer ${accessToken.value}`,
            'Content-Type': 'application/json'
          }
        });
        seasons.value = response.data.map(season => ({
          label: season.name,
          value: season.id
        }));
      } catch (error) {
        $q.notify({
          color: 'negative',
          message: 'Failed to load seasons. Please check your connection or try again later.',
          icon: 'warning'
        });
        console.error('Failed to fetch seasons:', error);
      }
    });

    // 当季节更改时调用该方法
    const onSeasonChange = (seasonId) => {
      const season = seasons.value.find(season => season.value === seasonId);
      console.log('Selected Season:', season); // 调试信息

      selectedSeason.value = seasonId;
      selectedSeasonName.value = season ? season.label : ''; // 使用 `label` 作为季节名称
      fetchFields(seasonId); // 调用 fetchFields 函数获取数据
    };

    // 获取并格式化字段数据
    const fetchFields = async (seasonId) => {
      if (!seasonId) return;
      try {
        const response = await axios.get(`https://34a97d79-460b-4dae-9ff7-1fdaa35a4031.mock.pstmn.io/api/v2/fields-service/seasons/${seasonId}/fields`, {
          headers: {
            Authorization: `Bearer ${accessToken.value}`,
            'Content-Type': 'application/json'
          }
        });
        
        console.log(response.data);
        console.log('Fields Data:', response.data);
        
        fieldsData.value = response.data.flatMap(field => {
          return field.contours.map((contour, index) => ({
            seasonId: selectedSeason.value,
            seasonName: selectedSeasonName.value,
            field_id: field.fieldId,
            field_name: index === 0 ? field.name : '',
            contour_id: contour.contourId,
            contour_name: contour.name,
            squareArea: contour.squareArea,
            crop_rotations: contour.cropRotations.map(rotation => ({
              cropRotationId: rotation.cropRotationId,
              culture: rotation.culture,
              sort: rotation.cultivar,
              start_date: rotation.startDate,
              end_date: rotation.endDate
            }))

          }));
        });
      } catch (error) {
        $q.notify({
          color: 'negative',
          message: 'Failed to load field data. Please check your connection or try again later.',
          icon: 'warning'
        });
        console.error('Failed to fetch fields by season:', error);
      }
    };

    // 跳转到编辑页面
    const navigateToEditPage = (contourId) => {
      const contourData = fieldsData.value.find(row => row.contour_id === contourId);
      
      console.log('Selected Contour Data:', contourData);

      if (contourData) {
        router.push({
          name: 'RotationPage',
          query: {
            seasonId: contourData.seasonId,
            seasonName: contourData.seasonName,
            fieldId: contourData.field_id,
            fieldName: contourData.field_name,
            contourId: contourData.contour_id,
            contourName: contourData.contour_name
          }
        });
      }
    };

    return {
      selectedSeason,
      selectedSeasonName,
      seasons,
      fieldsData,
      fieldsColumns,
      onSeasonChange,
      fetchFields,
      navigateToEditPage,
      pagination
    };
  }
};
</script>

<style scoped>
.q-pa-md {
  padding: 16px;
}

.q-card {
  background-color: #f9f9f9;
}

.q-card-section {
  padding: 16px;
}

.q-mt-md {
  margin-top: 16px;
}

.contour-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.edit-button {
  margin-left: 8px;
}

.crop-rotations {
  display: flex;
  flex-direction: row;
  gap: 16px; /* 控制多个作物之间的间距 */
}

.crop-rotation {
  margin-bottom: 8px;
}

.timeline {
  display: flex;
  align-items: center;
}

.timeline-dot {
  width: 8px;
  height: 8px;
  background-color: #2196F3;
  border-radius: 50%;
}

.timeline-line {
  flex: 1;
  height: 2px;
  background-color: #2196F3;
  margin: 0 8px;
}

.timeline-date {
  font-size: 0.85em;
}
</style>
