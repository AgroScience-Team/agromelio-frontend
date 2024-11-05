<template>
  <!-- seasons получать в onmounted, отображать список если массив не пустой -->
  <div class="season-field" v-if="seasonsList.length !==0">
    <div class="q-pa-md season-select">
      <q-btn-dropdown color="primary" label="Выбор сезона">
        <q-list>
          <q-item clickable v-close-popup @click="onItemClick">
            <q-item-section>
              <q-item-label>{{}}</q-item-label>
            </q-item-section>
          </q-item>
          <q-item clickable v-close-popup @click="onItemClick">
            <q-item-section>
              <q-item-label>Добавить сезон</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </q-btn-dropdown>
    </div>
    <!-- выбираем поле, появляется после того как выбрали сезон, и добавить вариант добавить поле -->
    <div class="q-pa-md field-select">
      <q-btn-dropdown color="primary" label="Выбор поля">
        <q-list>
          <q-item clickable v-close-popup @click="onItemClick">
            <q-item-section>
              <q-item-label>{{}}</q-item-label>
            </q-item-section>
          </q-item>

        </q-list>
      </q-btn-dropdown>
    </div>
  </div>
</template>
<script>
  export default {
    name: 'DropdownSeasonFieldButton',
    props: {
      seasonsList: {
        type: Array,
        required: true
      }
    },
    setup(props) {
      // const seasons = ref(props.seasonsList);
      // return{seasons}

    }
  }
</script>
<style scoped>
  .season-field {
    z-index: 1000;
    position: absolute;
    display: flex;
    /* justify-content: end; */
    flex-direction: row;
    top: 10px;
    right: 10px;
  }
</style>
