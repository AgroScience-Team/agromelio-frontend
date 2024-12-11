<template>
  <div class="map-container">
    <div id="map"></div>
    <!-- Диалог выбора цвета -->
    <q-dialog v-model="colorDialog" persistent>
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
    <!-- кнопки для добаления сезона/поля выпадающий список из сезонов/полей -->
    <dropdown-or-add-season-field-buttons @startDrawing="startDrawing"
     @removeSelectedPolygon="removeSelectedPolygon"
      @undoLastAction = "undoLastAction"></dropdown-or-add-season-field-buttons>
  </div>
</template>

<script>
import { ref, onMounted, watch } from "vue";
  import L from "leaflet";
import "leaflet/dist/leaflet.css";
import "leaflet-draw/dist/leaflet.draw.css";
import "leaflet-draw";
import { useRouter } from "vue-router";
import axios from "axios";
import { useQuasar } from "quasar";
import { userStore } from "src/usage";
// import MapPageEditButtons from "src/components/MapPageEditButtons.vue";
import * as turf from "@turf/turf";

import DropdownOrAddSeasonFieldButtons from "src/components/DropdownOrAddSeasonFieldButtons.vue";

export default {
  components: { DropdownOrAddSeasonFieldButtons },
  name: "MapComponent",
  setup() {
    const map = ref(null);
    const drawnItems = new L.FeatureGroup();
    const router = useRouter();
    const $q = useQuasar();
    const accessToken = userStore.state.access_token;
    const isDrawingEnabled = ref(false);
    let historyStack = [];  // Стек для хранения истории всех действий
    let polygon =  []; // массив для хранения точек полигона текущего
    const selectedSeason = ref(
      JSON.parse(sessionStorage.getItem('activeSeason')) || null
    );
    const selectedField = ref(
      JSON.parse(sessionStorage.getItem('activeField')) || null
    );
    const colorDialog = ref(false); // 控制颜色选择对话框显示 Управление отображением диалогового окна выбора цвета
    const selectedColor = ref("#ff0000"); // 默认选中的颜色 Выбранный по умолчанию цвет
    let currentLayer = null; // 当前绘制的多边形图层 Текущий слой нарисованных полигонов

    // Переход на страницу информации о поле
    const handlePopupClick = (fieldId) => {
      router.push(`/field_information?fieldId=${fieldId}`);
    };

    const fetchDataAndDrawPolygons = async () => {
      //Функция fetchDataAndDrawPolygons() загружает данные о полях (полигонах) из API и рисует их на карте:
      try {
        if (!accessToken) {
          console.error("No access token available");

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

        // Преобразование типов геометрии в читаемые и добавление метеданных
        for (const field of fields) {
          if (field.geom && field.geom.coordinates) {
            const coordinates = field.geom.coordinates.map((coord) => [
              coord.latitude,
              coord.longitude,
            ]);

// Полигоны и тайловые слои — важные компоненты картографии, особенно при работе с библиотеками, такими как Leaflet.
    // Полигон — это замкнутая многоугольная фигура на карте, которая создается из набора точек(координат).Он используется для выделения областей на карте, таких как земельные участки, зоны интереса или административные границы.
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
            // Добавление кнопки для информации о поле
            // Всплывающее окно: bindPopup() добавляет окно с информацией о поле и кнопкой для перехода на другую страницу.
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

    // Отмена выбора цвета
    const cancelColorSelection = () => {
      if (currentLayer) {
        drawnItems.removeLayer(currentLayer);
        currentLayer = null;
      }
      colorDialog.value = false;
    };
//     // Применить выбор цвета
//     Диалог выбора цвета: Пользователь может выбрать цвет, который применяется к текущему полигону.
// Установка стиля: Метод setStyle() обновляет цвет и прозрачность.
    const applyColorSelection = () => {
      if (currentLayer) {
        currentLayer.setStyle({
          color: selectedColor.value,
          fillColor: selectedColor.value,
          fillOpacity: 0.5,
        });
        drawnItems.addLayer(currentLayer);  // Убедитесь, что полигон добавляется в drawnItems
        currentLayer = null;
      }
      colorDialog.value = false;
      console.log("Selected color:", selectedColor.value);
    };

    // 初始化地图和绘制功能 Инициализация функций карты и черчения
    const initDrawing = () => {
      //Leaflet Draw позволяет пользователю рисовать новые полигоны:
      // const drawControl = new L.Control.Draw({
      //   draw: {
      //     polygon: true,
      //     polyline: false,
      //     rectangle: false,
      //     circle: false,
      //     marker: false,
      //     circlemarker: false,
      //   },

      //   edit: {
      //     featureGroup: drawnItems,
      //   },
      // });
      // map.value.addLayer(drawnItems);

      // map.value.addControl(drawControl);


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

      // draw: toolbarclosed — это событие, которое срабатывает, когда пользователь закрывает панель инструментов рисования в Leaflet Draw.
      map.value.on("draw:toolbarclosed", () => {

        drawControl._toolbars.draw.disable();
        map.value.off("mousemove"); // 停止鼠标移动事件监听 Прекратите прослушивать события движения мыши
      });

    }

    //при нажатии на кнопку добавления контура в DropdownOrAddSeasonFieldButtons, рисовать полигон
    let drawControl = null;
    let drawnHandler = null;  // Объявляем переменную для обработчика
    let selectedPolygon = null;  // Переменная для хранения выделенного полигона
    let tempLayer = null;
    const startDrawing = (isDrawing) => {
      isDrawingEnabled.value = isDrawing;
      console.log("isDrawing ",isDrawing);
      if (isDrawingEnabled.value) {
        if (!drawControl) {

          drawControl = new L.Draw.Polygon(map.value, {
            shapeOptions: { color: 'blue' }
              // shapeOptions: { color: 'transparent' },  // Устанавливаем цвет линий полигона как прозрачный

          });
          console.log("enabling drawControl");
        }
        drawControl.enable();  // Активирует режим рисования      }
        console.log('enable');

        if (!drawnHandler) {
          drawnHandler = (event) => {
            console.log("start of drawhandle");
            polygon = [];
            const layer = event.layer;
            // Добавляем обработчик клика на полигон
            // Обработчик клика на полигон для выделения
            layer.on('click', () => {
              // Устанавливаем стиль для нового выбранного полигона
              selectedPolygon = layer;
              selectedPolygon.setStyle({ weight: 8, fillOpacity: 0.2, color:'black' });  // Меняем цвет для выделения
            });
            console.log("map painting");
            if (!layer || !layer.getLatLngs || layer.getLatLngs().length === 0) {
              console.error("Invalid layer or empty coordinates");
              return;
            }

            const newPolygon = layer.toGeoJSON();
            const newPolygonColor = selectedColor.value;
            console.log("Координаты текущего нарисованного контура:", newPolygon.geometry.coordinates);


            let isOverlap = false;
            // При создании нового полигона проверяется, не пересекается ли он с существующими:

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
                // Закрываем полигоны перед проверкой

                if (turf.intersect(turfNew, turfExisting)) {
                  console.log("intersection");
                  isOverlap = true;
                }
                // if (turf.booleanIntersects(turfNew, turfExisting)) {
                //   isOverlap = true;
                // }
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
            console.log("Polygon coordinates:", layer.getLatLngs());

            // Проверка на существование и добавление слоя
            drawnItems.addLayer(layer);
            map.value.addLayer(drawnItems);  // Это также может потребоваться
            currentLayer = layer;

            colorDialog.value = true;
          }
          // Старт рисования полигона

          // Начало рисования полигона
          map.value.on('draw:drawstart', (e) => {
            console.log('draw:drawstart');
            if (e.layerType === 'polygon') {
              currentLayer = e.layer;  // Сохраняем текущий слой
              polygon = [];       // Очищаем массив точек
            }
          });

          // Добавление новой точки при рисовании
          map.value.on('draw:drawvertex', (e) => {
            console.log('draw:drawvertex');
            currentLayer = e.layer;  // Обновляем текущий полигон на этапе рисования
            console.log(e.layer);
            const latlng = e.latlng;       // Координаты новой точки
            polygon.push(latlng);     // Добавляем в массив
          });

          map.value?.on("draw:created", drawnHandler);
          // Слушаем клик на карте для добавления точек
          map.value?.on('click', function (e) {
            // historyStack.push([...polygon])
            // Добавляем точку в массив точек
            console.log("click");
            polygon.push([e.latlng.lat, e.latlng.lng]);
            console.log("length = ", polygon.length);
            // Полигон будет перерисовываться только после удаления или отмены
            // historyStack.push({ 'action': 'add_point', 'point': [e.latlng.lat, e.latlng.lng] });
            // if (tempLayer) {
            //   map.value.removeLayer(tempLayer);  // Удаляем старый временный слой
            // }
            // tempLayer = L.polygon(polygon, { color: 'blue' }).addTo(map.value);  // Создаем новый временный слой

          });
        }
      }
      else {
        console.log("else");
        if (drawControl) {
          console.log("Disabling drawControl");
          drawControl.disable();  // Отключаем режим рисования
          if (drawnHandler) {
            map.value?.off("draw:created", drawnHandler);
            drawnHandler = null;  // Сбрасываем обработчик
          }
        }
      }
    }

    const removeSelectedPolygon = () => {
      if (selectedPolygon) {
        historyStack.push({ action: 'delete_polygon', layer: selectedPolygon });

        drawnItems.removeLayer(selectedPolygon);  // Удаляем слой из группы
        map.value.removeLayer(selectedPolygon);   // Удаляем с карты
        // historyStack.push({
        //   action: 'delete',
        //   layer: selectedPolygon
        // });
        selectedPolygon = null;  // Сбрасываем выбранный полигон
        console.log("Selected polygon removed");
      } else {

        $q.notify({
          message: 'Выберите контур',
          type: 'negative'
        })
        console.log("No polygon selected to remove");
      }
    };
    // Функция для обновления полигона
    const updatePolygon = () => {
      console.log(currentLayer);
      if (currentLayer) {
        console.log("current layer exist");
        // Если полигон уже существует, обновляем его с новыми точками
        currentLayer.setLatLngs(polygon);
        // map.value.invalidateSize()

      }
      else {
        // Если полигон еще не завершен, перерисовываем черновой полигон
        const layers = drawnItems.getLayers();
        if (polygon.length > 0) {
          drawnItems.removeLayer(layers[layers.length - 1]);  // Удаляем последний слой
        }
        // if (polygon.length > 1) {
        //   if (tempLayer) {
        //     console.log('dell');
        //     map.value.removeLayer(tempLayer);  // Удаляем старый временный слой
        //   }
        //   tempLayer = L.polygon(polygonPoints, { color: 'blue' }).addTo(map.value);
        // }
      }
    };
    const undoLastAction = () => {
      console.log("undo last action");
      console.log(historyStack);
      if (drawControl && drawControl._markers && drawControl._markers.length > 0) {
        // Удаляем последнюю маркерную точку из массива
        console.log(drawControl);
        const lastMarker = drawControl._markers.pop();
        console.log(drawControl);
        drawControl._markerGroup.removeLayer(lastMarker);

        // Обновляем полигон на карте, удаляя последнюю точку
        const latlngs = drawControl._markers.map(marker => marker.getLatLng());

        // Обновляем полигон, только если есть больше двух точек (полигоны требуют 3)
        if (latlngs.length >= 3) {
          const latlngs = drawControl._markers.map(marker => marker.getLatLng());

          drawControl._poly.setLatLngs(latlngs);  // Применяем изменения к полигону
        } else {
          drawControl._poly.setLatLngs([]);  // Очистка, если точек недостаточно
        }
      }
      // if (polygon.length > 0) { // если есть текущий полигон который рисуем, то удаляем последнюю точку
      //   console.log("delete point");
      //   // polygon.pop();  // Удаляем последнюю точку
      //   // console.log(polygon);
      //   // // updatePolygon();        // Перерисовываем полигон
      //   // if (tempLayer) {
      //   //   map.value.removeLayer(tempLayer);  // Удаляем текущий временный полигон
      //   // }

      //   // if (polygon.length > 1) {
      //   //   tempLayer = L.polygon(polygon, { color: 'blue' }).addTo(map.value);  // Перерисовываем полигон без последней точки
      //   // }
      //   console.log("drawcontor");
      //   console.log(drawControl);
      //   // if (drawControl._drawing) {
      //   //   console.log("hereee");
      //   //   const lastVertex = drawControl._poly.getLatLngs().length - 1;
      //   //   if (lastVertex >= 0) {
      //   //     // Удаляем последнюю точку
      //   //     drawControl._poly.getLatLngs().pop();
      //   //     drawControl._poly.redraw();  // Перерисовываем полигон
      //   //   }
      //   // }
      //   // polygon.pop();
      //   // currentLayer.setLatLngs([polygon]);
      //   if (currentLayer) {
      //     console.log("start to update polygon");
      //     const latlngs = currentLayer.getLatLngs()[0];  // Получаем массив координат
      //       latlngs.pop();  // Удаляем последнюю точку
      //       currentLayer.setLatLngs([latlngs]);  // Обновляем полигон
      //   }
      // }

      else if (historyStack.length > 0) {

        const lastAction = historyStack.pop();  // Извлекаем последнее действие

        switch (lastAction.action) {
          // case 'save_field':
          //   console.log("delete point");
          //   polygon.pop();  // Удаляем последнюю точку
          //   console.log(polygon);
          //   updatePolygon();        // Перерисовываем полигон
          //   break;

          case 'delete_polygon':
            console.log("restore polygon");
            drawnItems.addLayer(lastAction.layer);  // Восстанавливаем удаленный полигон
            break;

          default:
            console.log("Неизвестное действие для отмены");
        }
      }
      else {
        console.log("нет действий для отмены");
      }
      // if (actionStack.length > 0) {
      //   const lastAction = actionStack.pop();  // Получаем последнее действие из стека

      //   if (!lastAction) {
      //     $q.notify({
      //       type: 'negative',
      //       message: 'Нет действий для отмены',
      //     });
      //     return;
      //   }

      //   if (lastAction.action === 'add') {
      //     // Если последнее действие - добавление точки, удалим ее и перерисуем полигон
      //     drawnItems.removeLayer(lastAction.layer);
      //     map.value.removeLayer(lastAction.layer);
      //     console.log('Polygon removed');
      //   } else if (lastAction.action === 'delete') {
      //     // Если последнее действие - удаление полигона, восстановим его
      //     drawnItems.addLayer(lastAction.layer);
      //     map.value.addLayer(lastAction.layer);
      //     console.log('Polygon restored');
      //   }
      // }
    }
    onMounted(() => {
      selectedSeason.value = JSON.parse(sessionStorage.getItem('activeSeason')) || null;
      selectedField.value = JSON.parse(sessionStorage.getItem('activeField')) || null;

      // Создание карты
      map.value = L.map("map").setView([59.420161, 30.01832], 15); //[широта, долгота], уровень_масштаба
      // Добавление тайлового слоя
      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution: "Map data &copy; OpenStreetMap contributors",
      }).addTo(map.value);

      map.value.addLayer(drawnItems);

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
      isDrawingEnabled,
      startDrawing,
      removeSelectedPolygon,
      undoLastAction
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

  .map-container {
    position: relative;
    height: 100vh;
    width: 100%;
  }

  #map {
    height: 100vh !important;
    width: 100% !important;
  }




  .leaflet-top.leaflet-left {
    top: 120px;
  }

  .popup-content {
    font-family: Arial, sans-serif;
    max-width: 300px;
    padding: 10px;
    background-color: #ffffff;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .popup-content .field-name {
    font-size: 18px;
    color: #333;
    margin-bottom: 5px;
  }

  .popup-content .crop-name {
    font-size: 16px;
    color: #666;
    margin-bottom: 10px;
  }

  .popup-content .field-description {
    font-size: 14px;
    color: #555;
    margin-bottom: 10px;
  }

  .popup-content .details-button {
    display: inline-block;
    padding: 8px 12px;
    margin-bottom: 10px;
    background-color: #007bff;
    color: #ffffff;
    text-align: center;
    border-radius: 5px;
    text-decoration: none;
    cursor: pointer;
  }

  .popup-content .details-button:hover {
    background-color: #0056b3;
  }

  .popup-content .meteo-data {
    margin-top: 10px;
  }

  .meteo-item {
    display: flex;
    align-items: center;
    font-size: 14px;
    color: #333;
    margin-bottom: 5px;
  }

  .meteo-icon {
    margin-right: 5px;
    color: #007bff;
  }

</style>
