
  <template>
    <q-page padding>
      <div class="q-pa-md q-gutter-md">
        <q-card>
          <q-card-section>
            <div class="text-h6 q-mb-md">Фильтрация по датам</div>
          </q-card-section>
          <q-separator />
          <q-card-section>
            <q-form>
              <q-input  
                v-model="startDate"
                label="Дата начала"
                outlined
                dense
                class="q-mb-md"
              >
                <template v-slot:append>
                  <q-icon name="event" class="cursor-pointer">
                    <q-popup-proxy ref="qDateProxyStart" transition-show="scale" transition-hide="scale">
                      <q-date v-model="startDate" @input="() => $refs.qDateProxyStart.hide()" />
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>

              <q-input
                v-model="endDate"
                label="Дата окончания"
                outlined
                dense
                class="q-mb-md"
              >
                <template v-slot:append>
                  <q-icon name="event" class="cursor-pointer">
                    <q-popup-proxy ref="qDateProxyEnd" transition-show="scale" transition-hide="scale">
                      <q-date v-model="endDate" :min="startDate" @input="() => $refs.qDateProxyEnd.hide()" />
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>
            </q-form>
          </q-card-section>
        </q-card>

        <q-card class="q-mt-md">
          <q-card-section>
            <div class="text-h6 q-mb-md">Таблица Севооборота</div>
          </q-card-section>
          <q-separator />
          <q-card-section>
            <q-table
              class="q-mt-md"
              virtual-scroll
              flat
              bordered
              v-model:pagination="pagination"
              :rows-per-page-options="[0]"
              :virtual-scroll-sticky-size-start="48"
              column-key="id"
              :rows="filteredRows"
              :columns="rotationColumns"
              row-key="id"
            >
              <template v-slot:body-cell-row_number="props">
                <q-td :props="props">
                  {{ props.rowIndex + 1 }}
                </q-td>
              </template>
              <template v-slot:body-cell-actions="props">
                <q-td :props="props">
                  <q-btn flat icon="delete" @click="deleteRow(props.row.id)" />
                </q-td>
              </template>
              <template v-slot:body-cell-edit="props">
                <q-td :props="props">
                  <q-btn flat icon="launch" @click="navigateToPage(props.row.id)" />
                </q-td>
              </template>
            </q-table>
          </q-card-section>
        </q-card>
      </div>
    </q-page>
  </template>
  <script>
  import { ref, computed, onMounted, reactive } from 'vue';
  import axios from 'axios';
  import { userStore } from 'src/usage';
  import { useQuasar } from 'quasar';
  import { useRouter } from 'vue-router';

  // Функция для преобразования строки формата dd-MM-yyyy в объект Date
  function parseDate(dateString) {
    const [day, month, year] = dateString.split('-');
    return new Date(`${year}-${month}-${day}`);
  }

  export default {
    setup() {
      const startDate = ref('');
      const endDate = ref('');
      const $q = useQuasar();
      const router = useRouter();

      const rotationData = reactive([]);
      const rotationColumns = reactive([
        { name: 'field_area', label: 'Название поля', align: 'center', field: 'field_area', sortable: true },
        { name: 'culture', label: 'Культура', align: 'center', field: 'culture', sortable: true },
        { name: 'start_time', label: 'Дата начала', align: 'center', field: 'start_time', sortable: true },
        { name: 'end_time', label: 'Дата окончания', align: 'center', field: 'end_time', sortable: true },
        { name: 'description', label: 'Описание', align: 'center', field: 'description' },
        { name: 'edit', label: 'Редактировать', align: 'center', field: row => row.id, format: val => `${val}` },
        { name: 'actions', label: 'Удалить', align: 'center', field: row => row.id, format: val => `${val}` }
      ]);

      const accessToken = userStore.state.access_token;

      async function deleteRow(rowId) {
        try {
          const response = await axios.delete(`http://smart.agromelio.ru/api/fields/crop-rotations?id=${rowId}`, {
            headers: {
              Authorization: `Bearer ${accessToken}`,
              'Content-Type': 'application/json'
            }
          });

          if (response.status === 204) {
            $q.notify({
              color: 'green-5',
              textColor: 'white',
              icon: 'check',
              message: 'Удалено'
            });
            rotationData.splice(rotationData.findIndex(row => row.id === rowId), 1);
          } else {
            console.error('Error deleting row:', response);
          }
        } catch (error) {
          console.error('Error during API call:', error);
        }
      }

      async function navigateToPage(rowId) {
        router.push({ path: '/fetch_rotation', query: { id: rowId } });
      }

      onMounted(async () => {
        const accessToken = userStore.state.access_token;

        if (!accessToken) {
          console.log('No access token available');
          $q.notify({
            type: 'negative',
            message: 'Залогиньтесь, пожалуйста'
          });
          return;
        }

        try {
          const response = await axios.get('http://smart.agromelio.ru/api/fields/crop-rotations/organization', {
            headers: {
              Authorization: `Bearer ${accessToken}`,
              'Content-Type': 'application/json'
            }
          });
          const data = response.data;
          console.log(data);

          if (data) {
            data.forEach(item => {
              rotationData.push({
                id: item.id,
                culture: item.crop.name,
                field_area: item.field.fieldName,
                start_time: item.startDate,
                end_time: item.endDate,
                description: item.description
              });
            });
          }
        } catch (error) {
          console.error('Wrong API', error);
        }
      });

      const filteredRows = computed(() => {
        return rotationData.filter(row => {
          const rowStart = parseDate(row.start_time);
          const rowEnd = parseDate(row.end_time);

          const startMatch = startDate.value ? parseDate(startDate.value) <= rowStart : true;
          const endMatch = endDate.value ? parseDate(endDate.value) >= rowEnd : true;

          return startMatch && endMatch;
        });
      });

      return {
        navigateToPage,
        deleteRow,
        rotationColumns,
        rotationData,
        startDate,
        endDate,
        filteredRows,
        pagination: ref({ rowsPerPage: 1000 })
      };
    }
  };
  </script>

  <style scoped>
  .q-pa-md {
    padding: 16px;
  }

  .q-gutter-md [class*=" q-"] {
    margin-bottom: 16px;
  }

  .q-card {
    background-color: #f9f9f9;
  }

  .q-card-section {
    padding: 16px;
  }

  .q-separator {
    margin: 16px 0;
  }

  .q-mt-md {
    margin-top: 16px;
  }

  .q-mb-md {
    margin-bottom: 16px;
  }
  </style>

