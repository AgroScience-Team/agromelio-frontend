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
          <q-table
            flat
            bordered
            :rows="fieldsData"
            :columns="fieldsColumns"
            row-key="contourId"
            v-model:pagination="pagination"
          >
            <template v-slot:body-cell-fieldName="props">
              <q-td :props="props">
                <div>{{ props.row.fieldName }}</div>
              </q-td>
            </template>

            <template v-slot:body-cell-contours="props">
              <q-td :props="props">
                <div class="contour-item" :style="{ color: '#' + props.row.contourColor }">
                  {{ props.row.contourName }} ({{ props.row.squareArea }} га)
                  <q-btn
                    flat
                    icon="edit"
                    color="primary"
                    @click="navigateToEditPage(props.row.contourId)"
                    class="edit-button"
                  />
                </div>
              </q-td>
            </template>

            <template v-slot:body-cell-cropRotations="props">
              <q-td :props="props">
                <div class="crop-rotations">
                  <div
                    v-for="crop in props.row.cropRotations"
                    :key="crop.cropRotationId"
                    class="crop-rotation"
                  >
                    <span>{{ crop.culture }} ({{ crop.cultivar }})</span>
                    <div class="timeline">
                      <span class="timeline-dot"></span>
                      <span class="timeline-line"></span>
                      <span class="timeline-date">{{ crop.startDate }}</span> - <span class="timeline-date">{{ crop.endDate }}</span>
                      <div class="rotation-description">{{ crop.description }}</div>
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

    const OnSeason = ref(null);
    const seasons = ref([]);
    const fieldsData = ref([]);
    const accessToken = computed(() => userStore.state.access_token);
    const pagination = ref({ rowsPerPage: 10 });


    const fieldsColumns = [
      { name: 'fieldName', label: 'поля', align: 'center', field: 'fieldName', style: 'width: 20%' },
      { name: 'contours', label: 'Контуры', align: 'center', field: 'contours', style: 'width: 30%' },
      { name: 'cropRotations', label: 'Посевы', align: 'center', field: 'cropRotations', style: 'width: 50%' }
    ];

    onMounted(async () => {
      try {
        const response = await axios.get(`${process.env.VUE_APP_BASE_URL}/api/v2/fields-service/seasons`, {
        // const response = await axios.get(`https://34a97d79-460b-4dae-9ff7-1fdaa35a4031.mock.pstmn.io/api/v2/fields-service/seasons`, {
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

    const fetchFields = async (OnSeason) => {
      if (!OnSeason || !OnSeason.value) return;
      const seasonId = OnSeason.value;
      const seasonName = OnSeason.label;
      try {
        // const response = await axios.get(`https://34a97d79-460b-4dae-9ff7-1fdaa35a4031.mock.pstmn.io/api/v2/fields-service/seasons/${seasonId}/fields`, {
          const response = await axios.get(`${process.env.VUE_APP_BASE_URL}/api/v2/fields-service/seasons/${seasonId}/fields`, {
          headers: {
            Authorization: `Bearer ${accessToken.value}`,
            'Content-Type': 'application/json'
          }
        });
        
        fieldsData.value = response.data.flatMap(field => {
          return field.contours.map((contour, index) => ({
            seasonId: seasonId,
            seasonName: seasonName,
            fieldId: field.id,
            fieldName: index === 0 ? field.name : '',
            // fieldDescription: index === 0 ? field.description : '',
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
      } catch (error) {
        $q.notify({
          color: 'negative',
          message: 'Failed to load field data. Please check your connection or try again later.',
          icon: 'warning'
        });
        console.error('Failed to fetch fields by season:', error);
      }
    };

    const navigateToEditPage = (contourId) => {
      const contourData = fieldsData.value.find(row => row.contourId === contourId);
      
      console.log('Selected Contour Data:', contourData);

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
  gap: 16px;
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
