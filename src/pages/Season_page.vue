<template>
  <q-page padding>
    <div class="q-pa-md q-gutter-md">
      <q-card>
        <q-card-section>
          <div class="text-h6">Севооборот</div>
        </q-card-section>
        <q-card-section>
          <div class="row justify-between items-center">
            <q-select
              v-model="OnSeason"
              label="Выбор сезона"
              :options="seasons"
              option-value="value"
              option-label="label"
              dense
              outlined
              class="season-select"
              @update:model-value="fetchFields"
            />
            <q-btn
              icon="add"
              label="Добавить сезон"
              color="primary"
              class="add-season-btn button-common"
              @click="navigateToAddSeason"
            />
          </div>
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

              <template v-slot:body-cell-fieldName="props">
                <q-td :props="props" class="field-name-cell">
                  {{ props.row.fieldName }}
                </q-td>
              </template>

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

    const navigateToAddSeason = () => {
      router.push({ name: 'add_season' });
    };

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
      navigateToEditPage,
      navigateToAddSeason
    };
  }
};
</script>


<style scoped>
.row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.season-select {
  width: 200px;
}
.button-common {
  width: 220px;
  height: 60px;
  background-color: #2e3a4b;
  color: #ffffff; 
  font-size: 1rem;
  border-radius: 10px; 
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.2); 
  text-transform: none; 
  padding: 0 16px; 
  font-weight: 500; 
  letter-spacing: 0.5px; 
  display: flex; 
  align-items: center; 
  justify-content: center; 
  gap: 6px; 
  transition: all 0.2s ease-in-out;
}


.table-scroll-container {
  width: 100%;
  overflow-x: auto;
  position: relative;
}

.fixed-table {
  table-layout: fixed;
  width: 100%;
  min-width: 1200px;
}

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

.fixed-table thead th.crop-rotations-header {
  position: sticky;
  left: 600px; 
  z-index: 3;
  background-color: #f5f5f5;
  text-align: left;
  padding-left: 20px; 
  transform: translateX(20px); 
}

.field-name-cell {
  width: 240px;
  min-width: 240px;
  text-align: center;
  vertical-align: middle;
  white-space: normal;
  padding: 12px;
}

.contours-cell {
  width: 360px;
  min-width: 360px;
  text-align: center;
  vertical-align: middle;
  white-space: normal;
  padding: 12px;
}

.crop-rotations-cell {
  min-width: 600px;
  width: auto;
  text-align: left;
  padding: 12px;
}

.crop-rotations {
  display: inline-flex;
  flex-wrap: nowrap;
  gap: 80px; 
  padding-right: 24px;
  align-items: center;
}

.crop-rotation-item {
  flex: 0 0 auto;
  min-width: 250px;
  text-align: center;
  position: relative;
  margin-bottom: 20px;
}

.crop-name {
  margin-bottom: 8px;
  white-space: nowrap;
  font-weight: 500;
  text-align: center;
}

.timeline {
  display: flex;
  align-items: center;
  position: relative;
  margin: 8px 0;
  padding: 0 20px;
  min-width: 200px;
}

.timeline-dot {
  width: 8px;
  height: 8px;
  background-color: #2196F3;
  border-radius: 50%;
  z-index: 2;
  flex-shrink: 0;
  margin-left: -3px;
  margin-right: -3px; 
}


.timeline-line {
  height: 2px;
  background-color: #2196F3;
  margin: 0 6px;
  min-width: 50px; 
  flex-grow: 1;
}

.timeline-date {
  font-size: 0.75em;
  background-color: white;
  padding: 2px 4px;
  border-radius: 2px;
  white-space: nowrap;
  position: absolute;
  top: 14px; 
  z-index: 3;
}


.timeline-date.left {
  transform: translateX(-50%);
}

.timeline-date.right {
  transform: translateX(50%);
  right: 0; 
}


.q-btn.flat {
  margin-left: 8px;
  padding: 4px;
}

.q-table {
  border: 1px solid rgba(0, 0, 0, 0.12);
}

.q-td {
  border-right: none;
}
</style>

