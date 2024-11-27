<template>
  <div class="map-container">
    <div id="map"></div>
    <!-- 颜色选择对话框 -->
    <!-- Диалог выбора цвета -->
    <q-dialog v-model="colorDialog" persistent >
      <q-card>
        <q-card-section>
          <div>
            <h5>Выберите цвет полигона</h5>
            <q-color v-model="selectedColor" />
          </div>
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="Отмена" @click="cancelColorSelection" />
          <q-btn flat label="Применить" color="primary" @click="applyColorSelection" />
        </q-card-actions>
      </q-card>
    </q-dialog>
    <q-btn
      fab
      color="positive"
      icon="add"
      class="add-field-button"
      @click="goToAddPage"
    />
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import L from "leaflet";
import "leaflet/dist/leaflet.css";
import "leaflet-draw/dist/leaflet.draw.css";
import "leaflet-draw";
import { useRouter } from "vue-router";
import axios from "axios";
import { useQuasar } from "quasar";
import { userStore } from "src/usage";
import * as turf from "@turf/turf";

export default {
  name: "MapComponent",
  setup() {
    const map = ref(null);
    const drawnItems = new L.FeatureGroup();
    const router = useRouter();
    const $q = useQuasar();
    const accessToken = userStore.state.access_token;

    const colorDialog = ref(false); // 控制颜色选择对话框显示 Управление отображением диалогового окна выбора цвета
    const selectedColor = ref("#ff0000"); // 默认选中的颜色 Выбранный по умолчанию цвет
    let currentLayer = null; // 当前绘制的多边形图层 Текущий слой нарисованных полигонов

    // Переход на страницу информации о поле
    const handlePopupClick = (fieldId) => {
      router.push(`/field_information?fieldId=${fieldId}`);
    };

    const fetchDataAndDrawPolygons = async () => {
      try {
        if (!accessToken) {
          $q.notify({
            type: "negative",
            message: "Залогиньтесь, пожалуйста",
          });
          return;
        }

        const response = await axios.get(
          `${process.env.VUE_APP_BASE_URL}/api/fields/organization/preview`,
          {
            headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
          }
        );

        const fields = response.data;

        for (const field of fields) {
          if (field.geom && field.geom.coordinates) {
            const coordinates = field.geom.coordinates.map((coord) => [
              coord.latitude,
              coord.longitude,
            ]);

            const polygon = L.polygon(coordinates, { color: `#${field.color}` }).addTo(map.value);
            const cropName = field.crop?.name || "";
            const popupContent = `
              <div class="popup-content">
              <strong class="field-name">${field.name}</strong><br>
              <strong class="crop-name">${cropName}</strong><br>
              <p class="field-description">${field.description}</p>
              <button id="popup-button-${field.id}" class="details-button">Посмотреть подробнее информацию</button>
              <div id="meteo-data-${field.id}" class="meteo-data">Загрузка...</div>
              </div>
            `;

            polygon.bindPopup(popupContent).on("popupopen", async (e) => {
              const popupButton = document.getElementById(`popup-button-${field.id}`);
              popupButton.addEventListener("click", () => handlePopupClick(field.id));

              // Получение метеоданных
              try {
                const meteoResponse = await axios.get(`${process.env.VUE_APP_BASE_URL}/api/meteo/preview/${field.id}`, {
                  headers: {
                    Authorization: `Bearer ${accessToken}`,
                    "Content-Type": "application/json",
                  },
                });
                const meteoData = meteoResponse.data;
                document.getElementById(`meteo-data-${field.id}`).innerHTML = `
                <div class="meteo-item"><i class="meteo-icon fas fa-thermometer-half"></i>Температура: ${meteoData.temperature} °C</div>
                <div class="meteo-item"><i class="meteo-icon fas fa-tint"></i>Влажность: ${meteoData.humidity} %</div>
                <div class="meteo-item"><i class="meteo-icon fas fa-wind"></i>Скорость ветра: ${meteoData.wind_speed} м/с</div>
                `;
              } catch (error) {
                console.error("Error fetching meteo data:", error);
                document.getElementById(`meteo-data-${field.id}`).innerHTML = " Нет метеоданных.";
              }
            });
          }
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    };

    // Переход к странице добавления поля
    const goToAddPage = () => {
      router.push("/add_field");
    };

    // 取消颜色选择 Отмена выбора цвета
    const cancelColorSelection = () => {
      if (currentLayer) {
        drawnItems.removeLayer(currentLayer);
        currentLayer = null;
      }
      colorDialog.value = false;
    };
    // 应用颜色选择 Применить выбор цвета
    const applyColorSelection = () => {
      if (currentLayer) {
        currentLayer.setStyle({
          color: selectedColor.value,
          fillColor: selectedColor.value,
          fillOpacity: 0.5,
        });
        currentLayer = null;
      }
      colorDialog.value = false;
      console.log("Selected color:", selectedColor.value);
    };

    // 初始化地图和绘制功能 Инициализация функций карты и черчения
    onMounted(() => {
      map.value = L.map("map").setView([59.420161, 30.01832], 15);

      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution: "Map data &copy; OpenStreetMap contributors",
      }).addTo(map.value);

      map.value.addLayer(drawnItems);

      const drawControl = new L.Control.Draw({
        draw: {
          polygon: true,
          polyline: false,
          rectangle: false,
          circle: false,
          marker: false,
          circlemarker: false,
        },
        edit: {
          featureGroup: drawnItems,
        },
      });
      map.value.addControl(drawControl);

      map.value.on("draw:created", (event) => {
        const layer = event.layer;

        if (!layer || !layer.getLatLngs || layer.getLatLngs().length === 0) {
          console.error("Invalid layer or empty coordinates");
          return;
        }

        const newPolygon = layer.toGeoJSON();
        const newPolygonColor = selectedColor.value;
        console.log("Координаты текущего нарисованного контура:", newPolygon.geometry.coordinates);

        let isOverlap = false;

        drawnItems.eachLayer((existingLayer) => {
          const existingPolygon = existingLayer.toGeoJSON();
          if (
            existingPolygon.geometry &&
            existingPolygon.geometry.type === "Polygon" &&
            newPolygon.geometry &&
            newPolygon.geometry.type === "Polygon"
          ) {
            const turfNew = turf.polygon(newPolygon.geometry.coordinates);
            const turfExisting = turf.polygon(existingPolygon.geometry.coordinates);

            if (turf.intersect(turfNew, turfExisting)) {
              isOverlap = true;
            }
          }
        });

        if (isOverlap) {
          $q.notify({
            type: "negative",
            message: "Контуры не должны перекрываться",
          });
          return;
        }

        layer.setStyle({
          color: newPolygonColor,
          fillColor: newPolygonColor,
          fillOpacity: 0.5,
        });

        drawnItems.addLayer(layer);
        currentLayer = layer;

        colorDialog.value = true;
      });

      map.value.on("draw:edited", (event) => {
        const layers = event.layers;
      
        layers.eachLayer((layer) => {
          const newLatLngs = layer.getLatLngs()[0]; 
      
          const coordinates = newLatLngs.map(latLng => [
            latLng.lng,
            latLng.lat
          ]);
      
          // 更新 GeoJSON 数据 Обновление данных GeoJSON
          const updatedPolygon = layer.toGeoJSON();
          updatedPolygon.geometry.coordinates = [coordinates]; 
      
          console.log("Обновленные координаты:", coordinates);
      
          // 保存颜色信息 Сохранение информации о цвете
          const currentStyle = layer.options;
      
          // 从 drawnItems 中移除旧图层 Удаление старых слоев из нарисованных элементов
          drawnItems.removeLayer(layer);
      
          // 创建新图层并应用样式 Создание нового слоя и применение стилей
          const newLayer = L.polygon(newLatLngs, currentStyle);
          drawnItems.addLayer(newLayer);
          
        });
      });
      
      map.value.on("draw:toolbarclosed", () => {

        drawControl._toolbars.draw.disable();
        map.value.off("mousemove"); // 停止鼠标移动事件监听 Прекратите прослушивать события движения мыши
      });

      // 加载已有多边形 Загрузка существующих полигонов
      fetchDataAndDrawPolygons();
    });

    return {
      map,
      goToAddPage,
      colorDialog,
      selectedColor,
      cancelColorSelection,
      applyColorSelection,
    };
  },
};
</script>


<style>
.map-container {
  position: relative;
  height: 100vh;
  width: 100%;
}

#map {
  height: 100vh !important;
  width: 100% !important;
}

.add-field-button {
  position: absolute;
  top: 20px;
  right: 20px;
  z-index: 1000;
}

.popup-content {
  font-family: Arial, sans-serif;
  max-width: 300px;
  padding: 10px;
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.details-button {
  background-color: #007bff;
  color: #fff;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
  border-radius: 5px;
}

.details-button:hover {
  background-color: #0056b3;
}
</style>
