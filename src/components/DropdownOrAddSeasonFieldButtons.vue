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
    @click="goToFieldPage">
    <div class="button-overlay">
      <p>Добавить поле</p>
    </div>
  </q-btn>
  <!-- seasons получаем в onmounted, отображать список если массив не пустой, иначе есть только кнопка добавления сезона -->
  <div class="season-field">
    <div v-if="seasonsList.length !==0" class="q-pa-md dropdown-button">
      <q-btn-dropdown color="primary" label="Выбор сезона">
        <q-list>
          <q-item v-for="season in filteredSeasons" :key="season.id" clickable v-close-popup
            @click="chooseActiveSeason(season)" dense
            :class="{ 'active-item': activeField && activeField.id === field.id }">
            <q-item-section>
              <q-item-label>{{season}}</q-item-label>
            </q-item-section>
          </q-item>
          <q-item v-if="!activeSeason" clickable v-close-popup @click="goToSeasonPage" dense
            class="add-season-field-item">
            <q-item-section>
              <q-item-label>ДОБАВИТЬ СЕЗОН</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </q-btn-dropdown>
    </div>
    <!-- выбираем поле, появляется после того как выбрали сезон и если полей нет, то отображается кнопка добавления поля -->
    <div v-if="fieldList.length !== 0" class="q-pa-md dropdown-button">
      <q-btn-dropdown color="primary" label="Выбор поля">
        <q-list>
          <q-item v-for="field in filteredFields" :key="field.id" clickable v-close-popup
            @click="chooseActiveField(field)" dense
            :class="{ 'active-item': activeField && activeField.id === field.id }">
            <q-item-section>
              <q-item-label>{{field}}</q-item-label>
            </q-item-section>
          </q-item>
          <q-item v-if="!activeField" v-close-popup @click="goToFieldPage" dense class="add-season-field-item">
            <q-item-section>
              <q-item-label>ДОБАВИТЬ ПОЛЕ</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </q-btn-dropdown>
    </div>
  </div>
</template>
<script>
  import MapPageEditButtons from 'src/components/MapPageEditButtons.vue';
  import { useRouter } from 'vue-router';
  import axios from 'axios';
  import { userStore } from "src/usage";

  import { ref, onMounted, computed } from "vue";
  import { useQuasar } from 'quasar';
  export default {
    name: 'DropdownOrAddSeasonFieldButtons',

    setup() {
      const router = useRouter();
      const $q = useQuasar();
      const accessToken = userStore.state.access_token;
      const activeSeason = ref();
      const activeField = ref();
      const seasonsList = ref([]); // массив сезонов
      const fieldList = ref([]); // массив полей
      const goToSeasonPage = () => {
        router.push("/add_season");
        fetchSeasons();
        //активный сезон тот который добавили
        activeSeason.value = seasonsList.value[seasonsList.value.length - 1];
      }
      const goToFieldPage = (id) => {
        router.push("/add_field/${id}");
        fetchFields();
        // делать активным поле которое сейчас добавили
        activeField.value = fieldList.value[fieldList.value.length - 1];
      }
      // если выбран активный сезон, то возвращать его, иначе весь список
      const filteredSeasons = computed(() => {
        return activeSeason.value ? [activeSeason.value] : seasonsList.value;
      });
      // если выбрано активное поле, то возвращать его, иначе весь список
      const filteredFields = computed(() => {
        return activeField.value ? [activeField.value] : fieldList.value;
      });
      const fetchSeasons = async () => {
        try {
          const response = await axios.get("http://localhost:9000/api/v2/fields-service/seasons", {
          // const response = await this.$api.get("/api/v2/fields-service/seasons", {
            headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
          });
          console.log(response.data);
        }
        catch (error) {
          console.log("Didn't get seasons");
        }

      }
      const fetchFields = async (id) => {
        try {
          // const response = await axios.get("http://localhost:8080/api/v2/fields-service/seasons/${id}/fields", {
          const response = await axios.get("http:///api/v2/fields-service/seasons/${id}/fields", {
          headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
          });
          console.log(response.data)
        }
        catch (error) {
          console.log("Didn't get fields");
        }

      }
      onMounted(async () => {
        fetchSeasons();
      }
      );
      const chooseActiveSeason = (season) => {
        activeSeason.value = season;
      }
      const chooseActiveField = (field) => {
        activeField.value = field;
      }
      return {
        goToSeasonPage, goToFieldPage, activeField, activeSeason, seasonsList, fieldList,
        chooseActiveSeason,
        chooseActiveField,
        filteredSeasons,
        filteredFields,
      }
    }
  }
</script>
<style scoped>
  .season-field {
    z-index: 1000;
    position: absolute;
    display: flex;
    flex-direction: row;
    top: 10px;
    right: 10px;
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

  .active-item{
    background-color: cadetblue;
    color: white;
  }

</style>
