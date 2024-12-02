<template>
  <!--  кнопка добавления сезона
   если хоть один сезон есть, скрыть -->
  <q-btn v-if="seasonsList.length === 0" fab color="primary" icon="add" class="add-button" @click="goToSeasonPage">
    <div class="button-overlay">
      <p>Добавить сезон</p>
    </div>
  </q-btn>
  <!-- если хоть одно поле есть, скрыть
   показывать после того как выбран сезон -->
  <q-btn v-if="fieldList.length === 0 && activeSeason" fab color="primary" icon="add" class="add-button"
    @click="goToFieldPage(activeSeason.id)">
    <div class="button-overlay">
      <p>Добавить поле</p>
    </div>
  </q-btn>
  <!-- seasons получаем в onmounted, отображать список если массив не пустой, иначе есть только кнопка добавления сезона
   если нет полей есть кнопка добавления поля -->
  <div class="season-field">
    <div v-if="seasonsList.length !==0" class="dropdown-button q-pa-md ">
      <q-btn-dropdown color="primary" label="Выбор сезона" persistent>
        <q-list>
          <q-item v-for="season in filteredSeasons" :key="season.id" clickable @click="chooseActiveSeason(season)" dense
            :class="{ 'active-item': activeSeason && activeSeason.id === season.id }">
            <!-- сделать чтобы класс работал и актив эл изменял цвет -->
            <q-item-section>
              <q-item-label>{{season.name}}</q-item-label>
            </q-item-section>
          </q-item>
          <q-item v-if="!activeSeason" clickable v-close-popup @click="goToSeasonPage()" dense
            class="add-season-field-item">
            <q-item-section>
              <q-item-label>ДОБАВИТЬ СЕЗОН</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </q-btn-dropdown>
    </div>
    <!-- выбираем поле, появляется после того как выбрали сезон и если полей нет, то отображается кнопка добавления поля -->
    <div v-if="fieldList.length !== 0" class="dropdown-button q-pa-md ">
      <q-btn-dropdown v-if="fieldList.length !== 0" color="primary" label="Выбор поля" persistent>
        <q-list>
          <q-item v-for="field in filteredFields" :key="field.id" clickable @click="chooseActiveField(field)" dense
            :class="{ 'active-item': activeField && activeField.id === field.id }">
            <q-item-section>
              <q-item-label>{{field.name}}</q-item-label>
            </q-item-section>
          </q-item>
          <q-item v-if="!activeField" v-close-popup @click="goToFieldPage()" dense class="add-season-field-item">
            <q-item-section>
              <q-item-label>ДОБАВИТЬ ПОЛЕ</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </q-btn-dropdown>
    </div>
  </div>
  <map-page-edit-buttons v-if="activeField && activeSeason" @startDrawing="startDrawing"
    @removeSelectedPolygon="removeSelectedPolygon" @undoLastAction="undoLastAction"></map-page-edit-buttons>
</template>
<script>
  import { useRouter, useRoute } from 'vue-router';
  import { userStore } from "src/usage";

  import axios from 'axios';
  import { ref, onMounted, computed } from "vue";
  import { useQuasar } from 'quasar';
  import MapPageEditButtons from './MapPageEditButtons.vue';
  export default {
    name: 'DropdownOrAddSeasonFieldButtons',
    components:{
      MapPageEditButtons
    },
    setup(props, { emit }) {
      const router = useRouter();
      const route = useRoute();
      const $q = useQuasar();
      const accessToken = userStore.state.access_token;
      // Получаем доступ к методам карты
      const mapRef = ref(null);
      const activeSeason = ref(null);

      const activeField = ref(null);
      const seasonsList = ref([]); // массив сезонов
      const fieldListAdded = ref([]); // массив полей добавленных но не отправленных на сервер
      // !!!!!!!!!!!если отправили на сервер поле, то удалять
      const fieldListSaved = ref([]); // массив полей полученных с сервера
      const fieldList = computed(() => {
        return fieldListAdded.value.concat(fieldListSaved.value);
      }); // массив полей
      const startDrawing = (isDrawing) => {
        emit('startDrawing', isDrawing);
}
      const removeSelectedPolygon = () => {
        emit('removeSelectedPolygon');
      }
      const undoLastAction = () => {
        emit('undoLastAction');
      }
      const goToSeasonPage = () => {
        console.log("Go to season page");
        router.push("/add_season");
        // делать активным сезон который сейчас добавили
      }
      const goToFieldPage = () => {
        console.log("Go to field page");
        router.push("/add_field");
        // делать активным поле которое сейчас добавили
      }
      // если выбран активный сезон, то возвращать его, иначе весь список
      const filteredSeasons = computed(() => {
        if (activeSeason.value) {
          return [activeSeason.value];
        }
        return seasonsList.value;
      });
      // если выбрано активное поле, то возвращать его, иначе весь список
      const filteredFields = computed(() => {
        if (activeField.value) {
          return [activeField.value];
        }
        return fieldList.value;
      });
      const fetchSeasons = async () => {
        try {
          const response = await axios.get(
            "https://295aeaa1-a948-4811-9198-0b73bcc777b9.mock.pstmn.io/api/v2/fields-service/seasons",
            {
            headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
          });
          console.log(response.data);
          seasonsList.value = response.data;
        }
        catch (error) {
          console.log("Didn't get seasons");
          seasonsList.value = [];

        }
      }
      const fetchFields = async (id) => {
        try {
          const response = await axios.get("http://localhost:9000/api/v2/fields-service/seasons/${id}/fields", {
          headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
          });
          console.log(response.data)
          fieldListSaved.value = response.data;
        }
        catch (error) {
          console.log("Didn't get fields");
        }

      }
      onMounted(async () => {
        if (sessionStorage.getItem("activeField")) {
          activeField.value = JSON.parse(sessionStorage.getItem("activeField"));
        }
        if (sessionStorage.getItem("activeSeason")) {
          activeSeason.value = JSON.parse(sessionStorage.getItem("activeSeason"));

          if (sessionStorage.getItem("fields")) {
            fieldListAdded.value = JSON.parse(sessionStorage.getItem("fields"));
          }
       }
        fetchSeasons();
        console.log("fields:" ,fieldList.value);

      });
      const chooseActiveSeason = (season) => {
        if (!activeSeason.value) {
          activeSeason.value = season;
          sessionStorage.setItem("activeSeason", JSON.stringify(activeSeason.value));
          // получаем список полей сезона
          fetchFields(activeSeason.value.id);
        }
        else {
          activeSeason.value = null;
          sessionStorage.removeItem("activeSeason");
          //очищаем добавленные но не отправленные на сервер сезоны и аактивный сезон
          if (activeField.value) {
            activeField.value = null;
            sessionStorage.removeItem("activeField");
            sessionStorage.removeItem("fields");
            fieldListAdded.value = [];
            fieldListSaved.value = [];
          }
        }
      }
      const chooseActiveField = (field) => {
        if (!activeField.value) {
          activeField.value = field;
          sessionStorage.setItem("activeField", JSON.stringify(activeField.value));
        }
        else {
          activeField.value = null;
          sessionStorage.removeItem("activeField");
        }
      }
      return {
        goToSeasonPage, goToFieldPage, activeField, activeSeason, seasonsList, fieldList,
        chooseActiveSeason,
        chooseActiveField,
        filteredSeasons,
        filteredFields,
        startDrawing,
        removeSelectedPolygon,
        undoLastAction
      }
    }
  }
</script>
<style scoped>
  .season-field {
    position: absolute;
    right: 10px;
    top: 10px;
    z-index: 1000;
    display: flex;
    flex-direction: row;
    gap: 20px;
  }

  .dropdown-button {
    position: relative;
    /* Выпадающий список позиционируется относительно кнопки */
    /* width: 100%; */
    /* Обеспечивает ширину контейнера */
    /* display: flex;
    flex-direction: column;
    align-items: flex-start;*/
  }

  .add-season-field-item {
    background-color: #222C3C;
    color: white;
  }

  .add-button {
    position: absolute;
    bottom: 60px;
    left: 10px;
    z-index: 1000;
  }

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

  .add-button:hover .button-overlay {
    display: block;
    width: 110px;
    height: 25px;
    padding-right: 10px;
    padding-left: 10px;
  }

  .active-item {
    background-color: rgb(152, 161, 182);
  }



</style>
