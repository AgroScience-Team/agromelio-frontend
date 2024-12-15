<template>
  <div class="all">
    <h3 class="season-creation"><strong>Создание сезона</strong></h3>
    <div v-show="formData" class="formingdata q-pa-md">
      <q-input v-model="formData.name" label="Название сезона" autogrow outlined class="q-mb-lg"></q-input>
      <div class="date-entering">
        <q-input v-model="formData.seasonStart" filled type="date" outlined hint="Дата начала сезона" class="q-mb-lg" />
        <q-input v-model="formData.seasonEnd" filled type="date" outlined hint="Дата конца сезона" class="q-mb-lg" />
      </div>
      <q-btn label="Готово" @click="submitData" :disabled="isSubmitDisabled" color="primary"
        class="full-width-button"></q-btn>
    </div>
  </div>
</template>

<script>
  import { ref, computed } from 'vue';
  import axios from 'axios';
  import { useQuasar } from 'quasar';
  import { useRouter } from 'vue-router';
  import { userStore } from 'src/usage';

  export default {
    name: 'AddSeasonPage',
    setup() {
      const router = useRouter();
      const accessToken = computed(() => userStore.state.access_token);
      
      // Form data
      const formData = ref({
        name: '',
        seasonStart: '',
        seasonEnd: ''
      });

      const $q = useQuasar();

      // Redirect to map page
      const goToMapPage = () => {
        router.push('/map');
      }

      // Disable submit button if fields are not filled
      const isSubmitDisabled = computed(() => {
        return !formData.value.name || !formData.value.seasonStart || !formData.value.seasonEnd;
      });

      // Submit form data
      const submitData = () => {
        // Validate if form data is complete
        if (isSubmitDisabled.value) {
          $q.notify({
            type: 'negative',
            message: 'Пожалуйста, заполните все поля'
          })
          return;
        };

        // Prepare request data
        const requestData = {
          season: 'SeasonBaseDTO',  // Add season field
          name: formData.value.name,
          startDate: formData.value.seasonStart,
          endDate: formData.value.seasonEnd,
        };

        console.log('Submitting data:', JSON.stringify(requestData));

        // Make the API call
        axios.post(`${process.env.VUE_APP_BASE_URL}/api/fields-service/season`, requestData, {
          headers: {
            'Authorization': `Bearer ${accessToken.value}`,
            'Content-Type': 'application/json'
          }
        })
        .then(response => {
          $q.notify({
            type: 'positive',
            message: 'Сезон успешно создан'
          })
          const seasonId = response.data.id;

          // Redirect to map page after creation
          goToMapPage();
        })
        .catch(error => {
          if (error.response && error.response.status === 500) {
            $q.notify({
              type: 'negative',
              message: 'Неизвестная ошибка'
            });
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
  .date-entering {
    line-height: 300px;
  }

  .all {
    display: flex;
    flex-direction: column;
    width: 500px;
    margin-left: 15px;
  }

  .season-creation {
    font-family: Verdana, Geneva, Tahoma, sans-serif;
  }

  .full-width-button {
    width: 100%;
  }
</style>
