<template>
  <!-- 3 кнопки будут видны если нажань на редактировать -->
  <div class="edit-mode q-pa-md">
      <q-btn v-if="editModeOn" fab color="primary" icon="undo" class="button">
        <div class="button-overlay">
          <p>Шаг назад</p>
        </div>
      </q-btn>
      <q-btn v-if="editModeOn" fab color="primary" icon="done" class="button" @click="goToAddPage">
        <div class="button-overlay">
          <p>Сохранить</p>
        </div>
      </q-btn>
      <q-btn v-if="editModeOn" fab color="primary" icon="delete" class="button" @click="confirm = true">
        <div class="button-overlay">
          <p>Удалить контур</p>
        </div>
      </q-btn>
      <!-- для подтверждения удаления
       сделать чтобы кнопочки да нет работали -->
      <q-dialog v-model="confirm" persistent>
        <q-card class="confirm-deleting q-pa-md" style="border-radius: 40px;">
          <q-card-section class="row items-center">
            <span align="center"><strong>Вы действительно хотите удалить этот объект?</strong></span>
          </q-card-section>
          <q-card-actions align="center">
            <q-btn label="Нет" color="primary" v-close-popup />
            <q-btn label="Да" color="primary" v-close-popup />
          </q-card-actions>
        </q-card>
      </q-dialog>
    <q-btn fab color="primary" icon="edit" class="button" @click="editModeOn = !editModeOn">
      <div class="button-overlay">
        <p>Режим редактирования</p>
      </div>
    </q-btn>
  </div>
</template>

<script>
  import { useQuasar } from 'quasar';
  import { ref, onMounted } from "vue";

  export default {
      name: 'MapPageEditButtons',
setup() {
      const $q = useQuasar();
      const confirm = ref(false);
      const editModeOn = ref(false);


      return {
        confirm,
        editModeOn
      };
    }
  }
</script>
<style scoped>
  .button-overlay {
    position: absolute;
    background-color: #222C3C;
    display: none;
    text-align: center;
    left: 90%;
    font-size: 10px;
    border-radius: 4px;
    font-family: Arial, sans-serif;
    white-space: nowrap;

  }

  .edit-mode {
    display: flex;
    z-index: 1000;
    position: absolute;
    flex-direction: column;
    bottom: 60px;
    left: 10px;
  }

  .button {
    margin: 5px
  }

  .button:hover .button-overlay {
    display: inline-block;
    /* width: 110px; */
    height: 25px;
    padding-right: 10px;
    padding-left: 10px;
    min-width: 50px;
    width: auto;
  }

  .confirm-deleting {
    height: 160px;
    width: 300px;
  }

</style>
