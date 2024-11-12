<template>
  <div class="all">
    <h3 class="field-creation"><strong>Создание поля</strong></h3>
    <div v-show="formData" class="formingdata q-pa-md">
      <q-input v-model="formData.name" label="Название поля" autogrow outlined class="q-mb-lg"></q-input>
      <q-input v-model="formData.name" label="Описание поля" autogrow outlined class="q-mb-lg"></q-input>
      <q-btn label="Готово" @click="submitData" :disabled="isSubmitDisabled" color="primary"
        class="full-width-button"></q-btn>
    </div>
  </div>
</template>
<script>
  import { ref, computed } from 'vue';
  import axios from 'axios';
  import * as turf from '@turf/turf';
  import { useQuasar } from 'quasar';
  import { useRouter, useRoute } from 'vue-router';
  import { userStore } from 'src/usage';

  export default {
    name: 'AddFieldPage',
    setup() {
      const router = useRouter();
      const route = useRoute();
      const seasonId = route.params.id;
      const formData = ref({
        name: '',
        seasonStart: '',
        seasonEnd: '',
        // id:''
      });

      const $q = useQuasar();

      const goToMapPage = () => {
        router.push('/map');
      }

      const isSubmitDisabled = computed(() => {
        return !formData.value.name || !formData.value.seasonStart || !formData.value.seasonEnd;
      })

      const accessToken = userStore.state.access_token;
      //click submit function
      const submitData = () => {
        //check if everydata is exsist
        if (!accessToken) {
          console.error('No access token available');
          $q.notify({
            type: 'negative',
            message: 'Залогиньтесь, пожалуйста'
          })
          return;
        }
        if (isSubmitDisabled.value) {
          $q.notify({
            type: 'negative',
            message: 'Пожалуйста, заполните все поля'
          })
          return;
        };

        console.log('success');
        console.log('Submitting data:', JSON.stringify(formData));
        // axios.post('http://smart.agromelio.ru/api/v2/fields-service/season', formData.value, {
        axios.post('/api/v2/fields-service/season', formData.value, {
          headers: {
            'Authorization': `Bearer ${accessToken}`,
            'Content-Type': 'application/json'
          }
        })
          .then(response => {
            $q.notify({
              type: 'positive',
              message: 'Сезон успешно создан'
            })
            const fieldId = response.data.id;

            // Роутинг: переход на страницу карты
            // router.push({ path: '/map', query: { fieldId: fieldId } });


            //куда все-таки надо перейти и нужно ли что-то делать с id сезона который приходит в респонсе????
            goToMapPage();

          })
          .catch(error => {
            if (error.response && error.response.status === 500) {
              $q.notify({
                type: 'negative',
                message: 'Неизвестная ошибка'
              })
            }
            console.error('Error submitting data', error);
          });

      };


      return {
        formData,
        submitData,
        isSubmitDisabled,
      };
    },
  };
</script>
<style>

  .all {
    display: flex;
    flex-direction: column;
    width: 500px;
    margin-left: 15px;
  }

  .field-creation {
    font-family: Verdana, Geneva, Tahoma, sans-serif;
  }

  .full-width-button {
    width: 100%;
  }

</style>

<!-- <template>
  <div>
    <div class="row q-gutter-sm buttons-container q-pa-md">
      <q-btn label="Загрузить файл" @click="uploadFile"></q-btn>
      <q-btn label="Добавить данные о почве" @click="goToSoilPage"></q-btn>
    </div>
    <input type="file" ref="fileInput" @change="handleFileUpload" style="display: none" />

    <div v-show="formData" class="form-container q-pa-md">
      <q-input v-model="formData.name" label="Имя поля" outlined dense class="q-mb-md"></q-input>
      <q-input v-model="formData.description" label="Описание" outlined dense class="q-mb-md"></q-input>
      <q-input v-model="formData.activityStart" label="Дата начала" hint="Format: DD-MM-YYYY" mask="##-##-####" outlined dense class="q-mb-md"></q-input>
      <q-input v-model="formData.activityEnd" label="Дата окончания" hint="Format: DD-MM-YYYY" mask="##-##-####" outlined dense class="q-mb-md"></q-input>

      <q-input v-model="colorInput" label="Выберите цвет" readonly @click="triggerColorPicker" outlined dense class="q-mb-md"></q-input>
      <input type="color" ref="hiddenColorPicker" style="display: none" @input="updateColorInput($event.target.value)" />

      <q-input v-model="coordinatesJSON" label="Координаты" outlined dense class="q-mb-md"></q-input>
      <q-card flat bordered class="q-ma-md">
        <q-card-section>
          <div class="text-h6">Площадь поля</div>
          <div class="text-subtitle2">{{ calculateArea ? calculateArea.toFixed(2) + ' square meters' : 'Нет координат' }}</div>
        </q-card-section>
      </q-card>

      <q-btn label="Готово" @click="submitData" :disabled="isSubmitDisabled" color="primary" class="full-width-button q-mt-md"></q-btn>
    </div>
  </div>
</template>


<script>
import { ref, computed } from 'vue';
import axios from 'axios';
import * as turf from '@turf/turf';
import { useQuasar } from 'quasar';
import { useRouter } from 'vue-router';
import { userStore } from 'src/usage';

export default {
  name: 'AddFieldPage',
  setup() {
    const router = useRouter();
    const fileInput = ref(null);
    const hiddenColorPicker = ref(null);
    const colorInput = ref('');
    const formData = ref({
      name: '',
      description: '',
      activityStart: '',
      activityEnd: '',
      geom: {
        type: '',
        coordinates: []
      },
      color: "000000",
      squareArea: ''
    });

    const $q = useQuasar();
    //hide color palette
    const triggerColorPicker = () => {
      hiddenColorPicker.value.click();
    };

    const updateColorInput = (color) => {
      colorInput.value = color.replace('#', '');
      formData.value.color = colorInput.value;
    };



    const goToSoilPage = () => {
      router.push('/add_soil');
    }

    const uploadFile = () => {
      fileInput.value.click();
    };
    //read KML file
    const handleFileUpload = async (event) => {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const text = e.target.result;
          parseKML(text, file);
        };
        reader.readAsText(file);
      }
    };

    const parseKML = (text, file) => {
      const parser = new DOMParser();
      const xmlDoc = parser.parseFromString(text, "text/xml");
      //read different key words
      const nameTags = xmlDoc.getElementsByTagName("name");
      if (nameTags.length > 0) {
        formData.value = formData.value || {};
        formData.value.name = nameTags[0].textContent;
      };

      const descriptionTags = xmlDoc.getElementsByTagName('description');
      if (descriptionTags.length > 0) {
        formData.value = formData.value || {};
        formData.value.description = descriptionTags[0].textContent;
      };

      const coordinatesTags = xmlDoc.getElementsByTagName("coordinates");
      if (coordinatesTags.length > 0) {
        const coordinatesRaw = coordinatesTags[0].textContent.trim();
        const coordinates = coordinatesRaw.split(/\s+/).map(coordStr => {
          const [longitude, latitude] = coordStr.split(',').map(Number);
          return { longitude, latitude };
        });

        formData.value = formData.value || {};
        formData.value.geom = {
          type: "Polygon",
          coordinates: coordinates
        };
      };
    }
    //turn geo into json
    const coordinatesJSON = computed({
      get() {
        return JSON.stringify(formData.value.geom.coordinates, null, 2);
      },
      set(value) {
        try {
          formData.value.geom.coordinates = JSON.parse(value);
        } catch (e) {
          $q.notify({
            type: 'negative',
            message: 'Неверный формат JSON'
          })
        }
      }
    });
    //calculate squared area
    const calculateArea = computed(() => {
      console.log(formData.value.geom.coordinates)
      if (!formData.value || !formData.value.geom || !formData.value.geom.coordinates || formData.value.geom.coordinates.length < 3) {
        return 0;
      }

      const coordinates = formData.value.geom.coordinates.map(coord => [coord.longitude, coord.latitude]);
      const polygon = turf.polygon([coordinates]);
      const area = turf.area(polygon);
      console.log(area);

      return area;
    });
    //turn time into dd-mm-yyyy
    const isValidDate = (dateStr) => {
      const regex = /^\d{2}-\d{2}-\d{4}$/;
      if (!regex.test(dateStr)) return false;

      const [day, month, year] = dateStr.split('-').map(Number);
      const date = new Date(year, month - 1, day);
      return date.getDate() === day && date.getMonth() === month - 1 && date.getFullYear() === year;
    };

    const isSubmitDisabled = computed(() => {
      return !formData.value.name || !formData.value.description || !formData.value.activityStart || !formData.value.activityEnd || formData.value.geom.coordinates.length === 0;
    });


    const accessToken = userStore.state.access_token;
    //click submit function
    const submitData = () => {
      //check if everydata is exsist
      if (!accessToken) {
        console.error('No access token available');
        $q.notify({
          type: 'negative',
          message: 'Залогиньтесь, пожалуйста'
        })
        return;
      }
      if (isSubmitDisabled.value) {
        $q.notify({
          type: 'negative',
          message: 'Пожалуйста, заполните все поля'
        })
        return;
      };
      if (!isValidDate(formData.value.activityStart)) {
        $q.notify({
          type: 'negative',
          message: 'Неверный формат даты начала активности.'
        })
        return;
      };
      if (!isValidDate(formData.value.activityEnd)) {
        $q.notify({
          type: 'negative',
          message: 'Неверный формат даты окончания действия.'
        })
        return;
      };

      formData.value.squareArea = calculateArea.value;
      formData.value.color = formData.value.color.replace('#', '');

      console.log('success');
      console.log('Submitting data:', JSON.stringify(formData)); //check data
      axios.post('http://smart.agromelio.ru/api/fields', formData.value, {
        headers: {
          'Authorization': `Bearer ${accessToken}`,
          'Content-Type': 'application/json'
        }
      })
        .then(response => {
          $q.notify({
            type: 'positive',
            message: 'Данные успешно отправлены'
          })
          const fieldId = response.data.id;

          router.push({ path: '/field_information', query: { fieldId: fieldId } });
        })
        .catch(error => {
          if (error.response && error.response.status === 409) {
            $q.notify({
            type: 'negative',
            message: 'Имя Поле уже существует'
          })
        }
          if (error.response && error.response.status === 500) {
            $q.notify({
            type: 'negative',
            message: 'Неизвестная ошибка'
          })
        }
          console.error('Error submitting data', error);
        });
    };


    return {
      fileInput,
      formData,
      uploadFile,
      handleFileUpload,
      parseKML,
      coordinatesJSON,
      calculateArea,
      submitData,
      isSubmitDisabled,
      goToSoilPage,
      hiddenColorPicker,
      colorInput,
      triggerColorPicker,
      updateColorInput
    };
  },
};
</script>

<style>

.buttons-container {
  margin-bottom: 16px;
}

.form-container {
  max-width: 600px;
  margin-left: 0;
}

.q-mb-md {
  margin-bottom: 16px;
}

.full-width-button {
  width: 100%;
}

</style> -->
