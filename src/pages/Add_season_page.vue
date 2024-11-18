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
      const formData = ref({
        name: '',
        seasonStart: '',
        seasonEnd: '',
      });

      const $q = useQuasar();

      const goToMapPage = () => {
        console.log(formData.value);
        router.push({
          path: '/map',
          query: { activeSeason: JSON.stringify(formData.value) }
        });
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
        console.log('Submitting data:', JSON.stringify(formData.value));
        // axios.post('http://smart.agromelio.ru/api/v2/fields-service/season', formData.value, {
        axios.post("https://295aeaa1-a948-4811-9198-0b73bcc777b9.mock.pstmn.io/api/v2/fields-service/season", formData.value, {
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
            formData.value.id = response.data.id;
            // переходим на страницу карты в квери передавая созданный сезон
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
