<template>
  <div class="map-container">
    <div id="map"></div>
    <!-- Диалог выбора цвета -->
    <q-dialog v-model="colorDialog" persistent>
      <q-card style="width: 300px">
        <q-card-section>
          <div>
            <q-input
              dense
              outlined
              v-model="contourName"
              label="Название контура"
              class="q-mb-md"
            />
          </div>
          <div>
            <text>Выберите цвет полигона:</text>
            <q-color v-model="selectedColor" />
          </div>
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat dense label="Отмена" @click="cancelColorSelection" />
          <q-btn
            flat
            dense
            label="Применить"
            color="primary"
            @click="applyColorSelection"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
    <!-- кнопки для добаления сезона/поля выпадающий список из сезонов/полей -->
    <dropdown-or-add-season-field-buttons
      @startDrawing="startDrawing"
      @removeSelectedPolygon="removeSelectedPolygon"
      @undoLastAction="undoLastAction"
      @postContours="postContours"
      @selectedField="updateSelectedField"
      :updateFields="updateFieldsInChild"
    ></dropdown-or-add-season-field-buttons>
  </div>
</template>

<script>
import { ref, onMounted, watch, onUnmounted, onBeforeUnmount, computed } from "vue";
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
    const activeField = ref(sessionStorage.getItem("activeField"));
    let deleteStack = []; // Стек для хранения истории всех действий
    let isBorderVisible = true; // Флаг для отслеживания состояния рамки
    const contourName = ref("");
    const isNameInvalid = computed(() => !contourName.value.trim());

    const selectedSeason = ref(
      JSON.parse(sessionStorage.getItem("activeSeason")) || null
    );
    const selectedField = ref(
      JSON.parse(sessionStorage.getItem("activeField")) || null
    );
    const colorDialog = ref(false); // 控制颜色选择对话框显示 Управление отображением диалогового окна выбора цвета
    const selectedColor = ref("#ff0000"); // 默认选中的颜色 Выбранный по умолчанию цвет
    let currentLayer = null; // 当前绘制的多边形图层 Текущий слой нарисованных полигонов

    // Переход на страницу информации о поле
    const handlePopupClick = (fieldId) => {
      router.push(`/field_information?fieldId=${fieldId}`);
    };

    /*const fetchDataAndDrawPolygons = async () => {
      console.log("etchDataAndDrawPolygons");
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

        for (const field of fields) {
          if (field.coordinates) {
            const coordinates = field.coordinates.map((coord) => [
              coord.latitude,
              coord.longitude,
            ]);

            // Создание полигона
            const polygon = L.polygon(coordinates, {
              color: `#${field.color}`, // Цвет обводки
              weight: 5, // Толщина рамки
              opacity: 1, // Прозрачность рамки
              fillOpacity: 0.3, // Прозрачность заливки
            }).addTo(map.value);

            // Добавление попапа с информацией
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
              const popupButton = document.getElementById(
                `popup-button-${field.id}`
              );
              popupButton.addEventListener("click", () =>
                handlePopupClick(field.id)
              );

              // Получение метеоданных
              try {
                const meteoResponse = await axios.get(
                  `${process.env.VUE_APP_BASE_URL}/api/meteo/preview/${field.id}`,
                  {
                    headers: {
                      Authorization: `Bearer ${accessToken}`,
                      "Content-Type": "application/json",
                    },
                  }
                );
                const meteoData = meteoResponse.data;
                document.getElementById(`meteo-data-${field.id}`).innerHTML = `
              <div class="meteo-item"><i class="meteo-icon fas fa-thermometer-half"></i>Температура: ${meteoData.temperature} °C</div>
              <div class="meteo-item"><i class="meteo-icon fas fa-tint"></i>Влажность: ${meteoData.humidity} %</div>
              <div class="meteo-item"><i class="meteo-icon fas fa-wind"></i>Скорость ветра: ${meteoData.wind_speed} м/с</div>
            `;
              } catch (error) {
                console.error("Error fetching meteo data:", error);
                document.getElementById(`meteo-data-${field.id}`).innerHTML =
                  " Нет метеоданных.";
              }
            });

            // Обработка кликов по полигону для изменения цвета рамки
            polygon.on("click", function () {
              const currentColor = polygon.options.color;
              const newColor =
                currentColor === "#3388ff" ? "#ff5733" : "#3388ff"; // Переключение между двумя цветами
              polygon.setStyle({ color: newColor }); // Изменение цвета рамки
            });
          }
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    };*/

    const clearPolygons = () => {
      console.log("очищаем");
      map.value.eachLayer((layer) => {
        if (layer instanceof L.Polygon) {
          // Проверяем, является ли слой полигоном
          map.value.removeLayer(layer);
        }
      });
    };
    const fetchDataAndDrawPolygons = async () => {
      console.log("fetchplogons");
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
          `${process.env.VUE_APP_BASE_URL}/api/fields-service/fields/${selectedField.value.id}/contours`,
          {
            headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
          }
        );
        console.log("contoures got ", response.data);
        const contours = response.data;
        contours.forEach(async (contour) => {
          const coordinates = contour.coordinates.map((coord) => [
            coord.latitude,
            coord.longitude,
          ]);

          // Устанавливаем цвет для полигона
          const polygonColor = `#${contour.color}`;
          // Создаем полигон и добавляем его на карту
          const polygon = L.polygon(coordinates, {
            color: polygonColor,
            fillColor: polygonColor,
            fillOpacity: 0.5,
          }).addTo(map.value);
          console.log("отрисовали контур");
          const popupContent = `
                                <div class="popup-content">
            <strong>${contour.name}</strong>  <br>
            <text>Принадлежность к полю:</text> <br>
             <strong>${contour.name}</strong>  <br>
               <div class="button-container">

            <button id="contour-info-${contour.id}" class="details-button">Смотреть<br>информацию<br> о контуре</button><br>
            <button id="field-info-${contour.id}" class="details-button">Смотреть<br>информацию<br> о поле</button><br>
              </div>
              <div id="meteo-data-${selectedField.value.id}" class="meteo-data">Загрузка...</div>
              </div>
            `;
          polygon.bindPopup(popupContent).on("popupopen", async (e) => {
            const contourInfoButton = document.getElementById(
              `contour-info-${contour.id}`
            );
            contourInfoButton.addEventListener("click", () =>
              handleContourInfoClick(contour.id)
            );

            const fieldInfoButton = document.getElementById(
              `field-info-${contour.id}`
            );
            fieldInfoButton.addEventListener("click", () =>
              handleFieldInfoClick(selectedField.value.id)
            );

            try {
              console.log(selectedField.value.id);
              const meteoResponse = await axios.get(
                `${process.env.VUE_APP_BASE_URL}/api/meteo/preview/${selectedField.value.id}`,
                {
                  headers: {
                    Authorization: `Bearer ${accessToken}`,
                    "Content-Type": "application/json",
                  },
                }
              );
              const meteoData = meteoResponse.data;
              document.getElementById(`meteo-data-${field.id}`).innerHTML = `
                <div class="meteo-item"><i class="meteo-icon fas fa-thermometer-half"></i>Температура: ${meteoData.temperature} °C</div>
                <div class="meteo-item"><i class="meteo-icon fas fa-tint"></i>Влажность: ${meteoData.humidity} %</div>
                <div class="meteo-item"><i class="meteo-icon fas fa-wind"></i>Скорость ветра: ${meteoData.wind_speed} м/с</div>
                `;
            } catch (error) {
              console.error("Error fetching meteo data:", error);
              document.getElementById(
                `meteo-data-${selectedField.value.id}`
              ).innerHTML = " Нет метеоданных.";
            }
          });
          // Запрос на получение метеоданных
        });
      } catch (error) {
        console.error("Error fetching contours data:", error);
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

    //     Диалог выбора цвета: Пользователь может выбрать цвет, который применяется к текущему полигону.
    const applyColorSelection = () => {
      if (!isNameInvalid()) {
        return; // Если имя пустое, не закрываем окно
      }

      if (currentLayer) {
        currentLayer.setStyle({
          color: selectedColor.value,
          fillColor: selectedColor.value,
          fillOpacity: 0.5,
        });
        currentLayer.feature.properties.name = contourName.value;
        drawnItems.addLayer(currentLayer); // Убедитесь, что полигон добавляется в drawnItems
        currentLayer = null;
      }
      colorDialog.value = false;
      contourName.value = null;
      console.log("Selected color:", selectedColor.value);
    };

    //при нажатии на кнопку добавления контура в DropdownOrAddSeasonFieldButtons, рисовать полигон
    let drawControl = null;
    let drawnHandler = null; // Объявляем переменную для обработчика
    let selectedPolygon = null; // Переменная для хранения выделенного полигона
    const startDrawing = (isDrawing) => {
      isDrawingEnabled.value = isDrawing;
      console.log("isDrawing ", isDrawing);
      if (isDrawingEnabled.value) {
        if (!drawControl) {
          drawControl = new L.Draw.Polygon(map.value, {
            shapeOptions: { color: "blue" },
          });
          console.log("enabling drawControl");
        }
        drawControl.enable(); // Активирует режим рисования      }
        console.log("enable");

        if (!drawnHandler) {
          drawnHandler = (event) => {
            console.log("start of drawhandle");
            polygon = [];
            const layer = event.layer;
            // добавляем чтобы можно было дать имя полигону
            layer.feature = layer.feature || { type: "Feature" };
            layer.feature.properties = layer.feature.properties || {};
            layer.feature.properties.name = ""; // Временное пустое имя
            // Добавляем обработчик клика на полигон
            layer.on("click", () => {
              // Устанавливаем стиль для нового выбранного полигона
              selectedPolygon = layer;
              isBorderVisible = !isBorderVisible;
              if (isBorderVisible) {
                // Убираем рамку
                selectedPolygon.setStyle({ weight: 0 });
              } else {
                // Добавляем рамку
                selectedPolygon.setStyle({
                  weight: 4,
                  color: "black",
                  opacity: 1,
                }); // Толщина и цвет рамки
              }
            });
            if (
              !layer ||
              !layer.getLatLngs ||
              layer.getLatLngs().length === 0
            ) {
              console.error("Invalid layer or empty coordinates");
              return;
            }

            const newPolygon = layer.toGeoJSON();
            const newPolygonColor = selectedColor.value;
            console.log(
              "Координаты текущего нарисованного контура:",
              newPolygon.geometry.coordinates
            );

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
                const turfExisting = turf.polygon(
                  existingPolygon.geometry.coordinates
                );
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
            map.value.addLayer(drawnItems); // Это также может потребоваться
            currentLayer = layer;

            colorDialog.value = true;
          };

          // Начало рисования полигона
          map.value.on("draw:drawstart", (e) => {
            console.log("draw:drawstart");
            if (e.layerType === "polygon") {
              currentLayer = e.layer; // Сохраняем текущий слой
              polygon = []; // Очищаем массив точек
            }
          });
          map.value?.on("draw:created", drawnHandler);
        }
      } else {
        console.log("else");
        if (drawControl) {
          console.log("Disabling drawControl");
          drawControl.disable(); // Отключаем режим рисования
          if (drawnHandler) {
            map.value?.off("draw:created", drawnHandler);
            drawnHandler = null; // Сбрасываем обработчик
          }
        }
      }
    };

    const removeSelectedPolygon = () => {
      if (selectedPolygon) {
        deleteStack.push(selectedPolygon);
        drawnItems.removeLayer(selectedPolygon); // Удаляем слой из группы
        map.value.removeLayer(selectedPolygon); // Удаляем с карты
        selectedPolygon = null; // Сбрасываем выбранный полигон
        console.log("Selected polygon removed");
      } else {
        $q.notify({
          message: "Выберите контур",
          type: "negative",
        });
        console.log("No polygon selected to remove");
      }
    };

    const undoLastAction = () => {
      console.log("undo last action");
      // если есть маркеры на карте удаляем последнюю точку
      if (
        drawControl &&
        drawControl._markers &&
        drawControl._markers.length > 0
      ) {
        console.log("delete last painted point");
        // Удаляем последнюю маркерную точку из массива
        const lastMarker = drawControl._markers.pop();
        drawControl._markerGroup.removeLayer(lastMarker);
        // Обновляем полигон на карте, удаляя последнюю точку
        const latlngs = drawControl._markers.map((marker) =>
          marker.getLatLng()
        );
        // Обновляем отрисовку полигона
        drawControl._poly.setLatLngs(latlngs);
      } else if (deleteStack.length > 0) {
        // Восстанавливаем полигон из стека
        const lastPolygon = deleteStack.pop();
        console.log("restore polygon");
        drawnItems.addLayer(lastPolygon); // Добавляем слой обратно в коллекцию слоёв
        map.value.addLayer(lastPolygon); // Добавляем полигон на картy
      } else {
        console.log("нет действий для отмены");
      }
    };
    const updateFieldsInChild = ref(false); //чтобы обновить поля в ребенке после того как они отправились на сервер
    const postContours = async () => {
      if (!accessToken) {
        console.error("No access token available");
        $q.notify({
          type: "negative",
          message: "Залогиньтесь, пожалуйста",
        });
        return;
      }

      // преобразовываем координаты и имена контуров в массив
      const contours = [];
      drawnItems.eachLayer((layer) => {
        if (layer instanceof L.Polygon) {
          const geoJson = layer.toGeoJSON();
          // Извлекаем координаты из GeoJSON
          let coordinates = geoJson.geometry.coordinates[0].map((coord) => ({
            longitude: coord[0], // lng
            latitude: coord[1], // lat
          }));
          contours.push({
            contour: "ContourBaseDTO",
            name: layer.feature.properties.name,
            color: layer.options.fillColor.replace("#", ""),
            squareArea: turf.area(geoJson).toFixed(2), // Площадь в квадратных метрах, округлена до двух знаков
            coordinates: coordinates,
          });
        }
      });
      console.log("contours ", contours);
      const fieldToPost = {
        name: JSON.parse(sessionStorage.getItem("activeField"))["name"],
        description: JSON.parse(sessionStorage.getItem("activeField"))[
          "description"
        ],
        field: "FieldDTO",
        contours: contours,
      };
      console.log("field: ", fieldToPost);

      try {
        const response = await axios.post(
          `${process.env.VUE_APP_BASE_URL}/api/fields-service/seasons/${
            JSON.parse(sessionStorage.getItem("activeSeason"))["id"]
          }/field`,
          fieldToPost,
          {
            headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
          }
        );
        console.log("Контура успешно отправлены:", response.data["id"]);

        //в fields в sessionStorage убираем отправленное поле
        const fields = JSON.parse(sessionStorage.getItem("fields") || "[]");
        const activeField = JSON.parse(
          sessionStorage.getItem("activeField") || "[]"
        ); // Пример activeField с id

        // Удаляем объект, где id совпадает с activeField.id
        const updatedFields = fields.filter(
          (field) => field["name"] !== activeField["name"]
        );
        // Сохраняем обновленный массив в sessionStorage и добавляем id к activeField
        sessionStorage.setItem("fields", JSON.stringify(updatedFields));
        sessionStorage.setItem(
          "activeField",
          JSON.stringify(
            Object.assign(JSON.parse(sessionStorage.getItem("activeField")), {
              id: response.data["id"],
            })
          )
        );
        $q.notify({
          type: "positive",
          message: "Контура успешно отправлены!",
        });
      } catch (error) {
        console.error("Ошибка при отправке контуров:", error);
        $q.notify({
          type: "negative",
          message: "Ошибка при отправке данных!",
        });
      }
      updateFieldsInChild.value = true;  // Переключаем состояние на true
  setTimeout(() => {
    eventTriggered.value = false;  // Переключаем обратно на false после задержки
  }, 1000);  // Задержка в 1 секунду
    };
    const updateSelectedField = () => {
      console.log("update selected field");
      selectedField.value = sessionStorage.getItem("activeField") ? JSON.parse(JSON.parse(sessionStorage.getItem("activeField"))) : null;
    };
    // Watch для обработки изменений activeField
    watch(selectedField, (newValue) => {
      if (newValue) {
        console.log("Рисуем полигоны для поля:", newValue);
        fetchDataAndDrawPolygons(); // Функция рисует полигоны
      } else {
        console.log("Удаляем полигоны с карты");
        clearPolygons(); // Функция удаляет полигоны
      }
    });

    onMounted(async () => {
      selectedSeason.value =
        JSON.parse(sessionStorage.getItem("activeSeason")) || null;
      selectedField.value =
        JSON.parse(sessionStorage.getItem("activeField")) || null;

      // Создание карты
      map.value = L.map("map").setView([59.420161, 30.01832], 15); //[широта, долгота], уровень_масштаба
      // Добавление тайлового слоя
      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution: "Map data &copy; OpenStreetMap contributors",
      }).addTo(map.value);

      map.value.addLayer(drawnItems);

      // 加载已有多边形 Загрузка существующих полигонов
      if (
        selectedSeason.value &&
        selectedField.value &&
        selectedField.value.id
      ) {
        fetchDataAndDrawPolygons();
        //если изменяется activeField sessionStorage тогда удаляются/рисуются полигоны
      }
    });

    return {
      map,
      contourName,
      goToAddPage,
      colorDialog,
      selectedColor,
      cancelColorSelection,
      applyColorSelection,
      isDrawingEnabled,
      startDrawing,
      removeSelectedPolygon,
      undoLastAction,
      postContours,
      updateSelectedField,
      updateFieldsInChild
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
  /*top: 120px;*/
}

.popup-content {
  font-family: Arial, sans-serif;
  width: 300px;
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

/*.popup-content .details-button {
  display: inline-block;
  padding: 8px 12px;
  margin-bottom: 10px;
  background-color: #007bff;
  color: #ffffff;
  text-align: center;
  border-radius: 5px;
  text-decoration: none;
  cursor: pointer;
}*/

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

.button-container {
  display: flex; /* Используем Flexbox для горизонтального расположения */
  gap: 5px; /* Расстояние между кнопками */
  height: 50px;
  justify-content: center; /* Центровка по горизонтали */
}

.details-button {
  font-size: 13px; /* Размер шрифта на кнопках */
  color: rgba(0, 0, 0, 0.726); /* Цвет текста на кнопках */
  padding-left: 10px;
  padding-left: 10px;
  width: 110px;
  font-weight: bold;

  background-color: #007bff; /* Цвет фона на кнопках */
  border: none; /* Убираем границу кнопки */
  padding: 4px; /* Отступы внутри кнопки */
  border-radius: 5px; /* Скругление углов кнопки */
  line-height: 1; /* Меньший междустрочный отступ */
}

.details-button:hover {
  background-color: #0056b3; /* Цвет фона кнопки при наведении */
}
</style>
