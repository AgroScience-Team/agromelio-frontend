<template>
  <q-page padding>
    <div class="q-pa-md q-gutter-md">
      <q-card>
        <q-card-section>
          <div class="text-h6">Севооборот</div>
        </q-card-section>
        <q-card-section>
          <q-select
            v-model="OnSeason"
            label="Выбор сезона"
            :options="seasons"
            option-value="value"
            option-label="label"
            dense
            outlined
            @update:model-value="fetchFields"
          />
        </q-card-section>
      </q-card>

      <q-card class="q-mt-md">
        <q-card-section>
          <div class="text-h6">Информация о культурах</div>
        </q-card-section>
        <q-card-section>
          <div class="table-scroll-container">
            <q-table
              flat
              bordered
              :rows="fieldsData"
              :columns="fieldsColumns"
              row-key="contourId"
              class="fixed-table"
            >
              <!-- 第一列 -->
              <template v-slot:body-cell-fieldName="props">
                <q-td :props="props" class="field-name-cell">
                  {{ props.row.fieldName }}
                </q-td>
              </template>

              <!-- 第二列 -->
              <template v-slot:body-cell-contours="props">
                <q-td :props="props" class="contours-cell">
                  <div :style="{ color: ensureColorFormat(props.row.contourColor) }">
                    {{ props.row.contourName }} ({{ props.row.squareArea }} га)
                    <q-btn
                      flat
                      icon="edit"
                      color="primary"
                      @click="navigateToEditPage(props.row.contourId)"
                    />
                  </div>
                </q-td>
              </template>

              <!-- 第三列 -->
              <template v-slot:body-cell-cropRotations="props">
                <q-td :props="props" class="crop-rotations-cell">
                  <div class="crop-rotations">
                    <div
                      v-for="crop in props.row.cropRotations"
                      :key="crop.cropRotationId"
                      class="crop-rotation-item"
                    >
                      <div class="crop-name">{{ crop.culture }} ({{ crop.cultivar }})</div>
                      <div class="timeline">
                        <span class="timeline-date left">{{ formatDate(crop.startDate) }}</span>
                        <span class="timeline-dot"></span>
                        <span
                          class="timeline-line"
                          :style="{ width: calculateTimelineWidth(crop.startDate, crop.endDate) }"
                        ></span>
                        <span class="timeline-dot"></span>
                        <span class="timeline-date right">{{ formatDate(crop.endDate) }}</span>
                      </div>
                    </div>
                  </div>
                </q-td>
              </template>
            </q-table>
          </div>
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

    const OnSeason = ref(null);
    const seasons = ref([]);
    const fieldsData = ref([]);
    const accessToken = computed(() => userStore.state.access_token);

    const fieldsColumns = [
      { name: 'fieldName', label: 'поля', align: 'center', field: 'fieldName'},
      { name: 'contours', label: 'Контуры', align: 'center', field: 'contours'},
      { name: 'cropRotations', label: 'Посевы', align: 'center', field: 'cropRotations'}
    ];

    // fake data
    const fakeData = [
      {
        id: "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        name: "Поле 1",
        description: "Описание поля 1",
        contours: [
          {
            name: "Контур 1",
            color: "d1E6dF",
            id: "3fa85f64-5717-4562-b3fc-2c963f66afa6",
            squareArea: "10",
            cropRotations: [
              {
                id: "1",
                culture: "Культура 1",
                cultivar: "Сорт 1",
                startDate: "2024-07-01",
                endDate: "2024-07-21",
                description: "Описание 1"
              },
              {
                id: "2",
                culture: "Культура 2",
                cultivar: "Сорт 2",
                startDate: "2024-07-22",
                endDate: "2024-12-30",
                description: "Описание 2"
              }
            ]
          },
          {
            name: "Контур 2",
            color: "A1B2C3",
            id: "3fa85f64-5717-4562-b3fc-2c963f66afa7",
            squareArea: "20",
            cropRotations: [
              {
                id: "3",
                culture: "Культура 3",
                cultivar: "Сорт 3",
                startDate: "2024-08-01",
                endDate: "2024-08-15",
                description: "Описание 3"
              },
              {
                id: "4",
                culture: "Культура 4",
                cultivar: "Сорт 4",
                startDate: "2024-08-16",
                endDate: "2024-08-31",
                description: "Описание 4"
              }
            ]
          }
        ]
      },
      {
        id: "3fa85f64-5717-4562-b3fc-2c963f66afa8",
        name: "Поле 2",
        description: "Описание поля 2",
        contours: [
          {
            name: "Контур 3",
            color: "FF5733",
            id: "3fa85f64-5717-4562-b3fc-2c963f66afa9",
            squareArea: "30",
            cropRotations: [
              {
                id: "5",
                culture: "Культура 5",
                cultivar: "Сорт 5",
                startDate: "2024-09-01",
                endDate: "2024-09-15",
                description: "Описание 5"
              },
              {
                id: "6",
                culture: "Культура 6",
                cultivar: "Сорт 6",
                startDate: "2024-09-16",
                endDate: "2024-09-30",
                description: "Описание 6"
              }
            ]
          },
          {
            name: "Контур 4",
            color: "33FF57",
            id: "3fa85f64-5717-4562-b3fc-2c963f66afaa",
            squareArea: "40",
            cropRotations: [
              {
                id: "7",
                culture: "Культура 7",
                cultivar: "Сорт 7",
                startDate: "2024-10-01",
                endDate: "2024-10-20",
                description: "Описание 7"
              },
              {
                id: "8",
                culture: "Культура 8",
                cultivar: "Сорт 8",
                startDate: "2024-10-21",
                endDate: "2024-10-31",
                description: "Описание 8"
              },
              {
                id: "9",
                culture: "Культура 9",
                cultivar: "Сорт 9",
                startDate: "2024-10-01",
                endDate: "2024-10-20",
                description: "Описание 9"
              },
              {
                id: "10",
                culture: "Культура 10",
                cultivar: "Сорт 10",
                startDate: "2024-10-21",
                endDate: "2024-10-31",
                description: "Описание 10"
              },
              {
                id: "11",
                culture: "Культура 11",
                cultivar: "Сорт 11",
                startDate: "2024-10-11",
                endDate: "2024-10-20",
                description: "Описание 11"
              },
              {
                id: "12",
                culture: "Культура 12",
                cultivar: "Сорт 12",
                startDate: "2024-10-21",
                endDate: "2024-10-31",
                description: "Описание 12"
              },
              {
                id: "13",
                culture: "Культура 13",
                cultivar: "Сорт 13",
                startDate: "2024-10-01",
                endDate: "2024-11-20",
                description: "Описание 13"
              },
              {
                id: "14",
                culture: "Культура 14",
                cultivar: "Сорт 14",
                startDate: "2024-10-21",
                endDate: "2024-12-31",
                description: "Описание 14"
              }
              
            ]
          }
        ]
      },
      {
        id: "3fa85f64-5717-4562-b3fc-2c963f66afab",
        name: "Поле 3",
        description: "Описание поля 3",
        contours: [
          {
            name: "Контур 5",
            color: "5733FF",
            id: "3fa85f64-5717-4562-b3fc-2c963f66afac",
            squareArea: "50",
            cropRotations: [
              {
                id: "9",
                culture: "Культура 9",
                cultivar: "Сорт 9",
                startDate: "2024-11-01",
                endDate: "2024-11-15",
                description: "Описание 9"
              },
              {
                id: "10",
                culture: "Культура 10",
                cultivar: "Сорт 10",
                startDate: "2024-11-16",
                endDate: "2024-11-30",
                description: "Описание 10"
              }
            ]
          }
        ]
      }
    ];


    onMounted(async () => {
      try {
        const response = await axios.get(`${process.env.VUE_APP_BASE_URL}/api/fields-service/seasons`, {
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
      }
    });

    const fetchFields = async (OnSeason) => {
      if (!OnSeason || !OnSeason.value) return;
      const seasonId = OnSeason.value;
      const seasonName = OnSeason.label;
      try {
        const response = await axios.get(`${process.env.VUE_APP_BASE_URL}/api/fields-service/seasons/${seasonId}/fields`, {
          headers: {
            Authorization: `Bearer ${accessToken.value}`,
            'Content-Type': 'application/json'
          }
        });

        const data = response.data;

        // 如果数据为空或结构不符合预期，使用假数据
        if (!data || !Array.isArray(data) || data.length === 0) {
          fieldsData.value = fakeData.flatMap(field => {
            return field.contours.map((contour, index) => ({
              seasonId: "mock-season-id",
              seasonName: "Летний сезон",
              fieldId: field.id,
              fieldName: index === 0 ? field.name : '',
              contourId: contour.id,
              contourName: contour.name,
              contourColor: contour.color,
              squareArea: contour.squareArea,
              cropRotations: contour.cropRotations.map(rotation => ({
                cropRotationId: rotation.id,
                culture: rotation.culture,
                cultivar: rotation.cultivar,
                startDate: rotation.startDate,
                endDate: rotation.endDate,
                description: rotation.description
              }))
            }));
          });
        } else {
          // 正常加载数据
          fieldsData.value = data.flatMap(field => {
            return field.contours.map((contour, index) => ({
              seasonId: seasonId,
              seasonName: seasonName,
              fieldId: field.id,
              fieldName: index === 0 ? field.name : '',
              contourId: contour.id,
              contourName: contour.name,
              contourColor: contour.color,
              squareArea: contour.squareArea,
              cropRotations: contour.cropRotations.map(rotation => ({
                cropRotationId: rotation.id,
                culture: rotation.culture,
                cultivar: rotation.cultivar,
                startDate: rotation.startDate,
                endDate: rotation.endDate,
                description: rotation.description
              }))
            }));
          });
        }
      } catch (error) {
        $q.notify({
          color: 'negative',
          message: 'Failed to load field data. Please check your connection or try again later.',
          icon: 'warning'
        });

        // 在请求失败时也使用假数据
        fieldsData.value = fakeData.flatMap(field => {
          return field.contours.map((contour, index) => ({
            seasonId: "mock-season-id",
            seasonName: "Летний сезон",
            fieldId: field.id,
            fieldName: index === 0 ? field.name : '',
            contourId: contour.id,
            contourName: contour.name,
            contourColor: contour.color,
            squareArea: contour.squareArea,
            cropRotations: contour.cropRotations.map(rotation => ({
              cropRotationId: rotation.id,
              culture: rotation.culture,
              cultivar: rotation.cultivar,
              startDate: rotation.startDate,
              endDate: rotation.endDate,
              description: rotation.description
            }))
          }));
        });
      }
    };

    const calculateTimelineWidth = (startDate, endDate) => {
      const start = new Date(startDate);
      const end = new Date(endDate);
      const totalDays = (end - start) / (1000 * 60 * 60 * 24); // Calculate days
      const pixelPerDay = 10; // Scale factor: 10px per day
      return `${totalDays * pixelPerDay}px`;
    };

    const formatDate = (date) => {
      const options = { year: 'numeric', month: '2-digit', day: '2-digit' };
      return new Date(date).toLocaleDateString('ru-RU', options);
    };

    const ensureColorFormat = (color) => {
      return color.startsWith('#') ? color : `#${color}`;
    };

    const navigateToEditPage = (contourId) => {
      const contourData = fieldsData.value.find(row => row.contourId === contourId);
      if (contourData) {
        router.push({
          name: 'RotationPage',
          query: {
            seasonId: contourData.seasonId,
            seasonName: contourData.seasonName,
            fieldId: contourData.fieldId,
            fieldName: contourData.fieldName,
            contourId: contourData.contourId,
            contourName: contourData.contourName
          }
        });
      }
    };

    return {
      seasons,
      OnSeason,
      fieldsData,
      fieldsColumns,
      fetchFields,
      calculateTimelineWidth,
      formatDate,
      ensureColorFormat,
      navigateToEditPage
    };
  }
};
</script>


<style scoped>
/* 表格容器 */
.table-scroll-container {
  width: 100%;
  overflow-x: auto; /* 启用横向滚动 */
  position: relative;
}

/* 表格基础样式 */
.fixed-table {
  table-layout: fixed; /* 固定布局 */
  width: 100%;
  min-width: 1200px; /* 确保表格至少有足够的宽度 */
}

/* 表头样式 */
.fixed-table thead th {
  background-color: #f5f5f5;
  font-weight: bold;
  position: sticky;
  top: 0;
  z-index: 2;
  text-align: center;
  white-space: nowrap;
  padding: 12px;
}

/* 第三列表头保持左对齐并向右移动 */
.fixed-table thead th.crop-rotations-header {
  position: sticky;
  left: 600px; /* 假设前两列的总宽度为600px，这样保证第三列的表头保持在右侧 */
  z-index: 3;
  background-color: #f5f5f5;
  text-align: left; /* 左对齐 */
  padding-left: 20px; /* 向右移动一定的距离 */
  transform: translateX(20px); /* 可以使用transform调整进一步细化位置 */
}

/* 第一列 - 固定宽度 */
.field-name-cell {
  width: 240px;
  min-width: 240px;
  text-align: center;
  vertical-align: middle;
  white-space: normal;
  padding: 12px;
}

/* 第二列 - 固定宽度 */
.contours-cell {
  width: 360px;
  min-width: 360px;
  text-align: center;
  vertical-align: middle;
  white-space: normal;
  padding: 12px;
}

/* 第三列 - 基础宽度 + 可扩展 */
.crop-rotations-cell {
  min-width: 600px;
  width: auto;
  text-align: left;
  padding: 12px;
}

/* 作物轮作容器 */
.crop-rotations {
  display: inline-flex;
  flex-wrap: nowrap;
  gap: 80px; /* 增加作物之间的间距 */
  padding-right: 24px;
  align-items: center;
}

/* 作物轮作项目 */
.crop-rotation-item {
  flex: 0 0 auto;
  min-width: 250px;
  text-align: center;
  position: relative;
  margin-bottom: 20px; /* 增加作物项目之间的垂直间隔 */
}

/* 作物名称 */
.crop-name {
  margin-bottom: 8px;
  white-space: nowrap;
  font-weight: 500;
  text-align: center;
}

/* 时间线容器 */
.timeline {
  display: flex;
  align-items: center;
  position: relative;
  margin: 8px 0;
  padding: 0 20px;
  min-width: 200px;
}

/* 时间线点 */
.timeline-dot {
  width: 8px;
  height: 8px;
  background-color: #2196F3;
  border-radius: 50%;
  z-index: 2;
  flex-shrink: 0;
  margin-left: -3px; /* 将点与时间线的距离拉近 */
  margin-right: -3px; /* 将点与时间线的距离拉近 */
}

/* 时间线 */
.timeline-line {
  height: 2px;
  background-color: #2196F3;
  margin: 0 6px; /* 减少时间点和线之间的距离 */
  min-width: 50px; /* 保证线的最小宽度 */
  flex-grow: 1;
}

/* 时间日期标签 */
.timeline-date {
  font-size: 0.75em;
  background-color: white;
  padding: 2px 4px;
  border-radius: 2px;
  white-space: nowrap;
  position: absolute;
  top: 14px; /* 保证时间线点下方一定距离 */
  z-index: 3;
}

/* 开始日期 */
.timeline-date.left {
  transform: translateX(-50%);
}

/* 结束日期 */
.timeline-date.right {
  transform: translateX(50%);
  right: 0; /* 保证结束时间显示在正确位置 */
}

/* 编辑按钮样式 */
.q-btn.flat {
  margin-left: 8px;
  padding: 4px;
}

/* 确保表格边框正确显示 */
.q-table {
  border: 1px solid rgba(0, 0, 0, 0.12);
}

/* 删除表格中竖着的线 */
.q-td {
  border-right: none;
}
</style>

