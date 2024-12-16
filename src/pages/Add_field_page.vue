<template>
  <div class="all">
    <h3 class="field-creation"><strong>Создание поля</strong></h3>
    <div v-show="formData" class="formingdata q-pa-md">
      <q-input v-model="formData.name" label="Название поля" autogrow outlined class="q-mb-lg"></q-input>
      <q-input v-model="formData.description" label="Описание поля" autogrow outlined class="q-mb-lg"></q-input>
      <q-btn label="Готово" @click="submitData" :disabled="isSubmitDisabled" color="primary"
        class="full-width-button"></q-btn>
    </div>
  </div>
</template>
<script>
  import { ref, computed } from 'vue';
  import axios from 'axios';
  import { useQuasar } from 'quasar';
  import { useRouter, useRoute } from 'vue-router';
  import { userStore } from 'src/usage';
  import { defineEmits } from "vue";

  export default {
    name: 'AddFieldPage',
    setup() {
      const router = useRouter();
      const route = useRoute();
      const seasonId = route.params.id;
      const formData = ref({
        name: '',
        description: '',
      });

      const $q = useQuasar();

      const isSubmitDisabled = computed(() => {
        return !formData.value.name || !formData.value.description;
      })
      const goToMapPage = () => {
        console.log(formData.value);
        //сохраняем созданное поле в хранилище, в нем хранятся поля созданные, но не отправленные на сервер, тюк нужно отрисовать контура
        // Получаем существующий массив
        let existingArray = JSON.parse(sessionStorage.getItem("fields")) || [];

        // Добавляем новое значение в массив
        existingArray.push(formData.value);

        // Сохраняем обновленный массив
        sessionStorage.setItem("fields", JSON.stringify(existingArray));
        sessionStorage.setItem("activeField", JSON.stringify(formData.value));

        router.push({
          path: '/map',
        });
      }
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
        console.log('Saving data:', JSON.stringify(formData.value));

        $q.notify({
          type: 'positive',
          message: 'Поле успешно создано'
        });
        //переходим на страницу карты передавая в квери formData
        goToMapPage();
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
