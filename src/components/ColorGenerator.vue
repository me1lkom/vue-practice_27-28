<template>
  <div class="page">
    <header>
      <h1>Генератор цветов</h1>
    </header>
    <div class="content">
      <div class="color__actions">
        <div class="color-control colorCount">
          <label>Количество цветов: </label>
          <select v-model="colorCount" class="color__select" @click="">
            <option value="3">3</option>
            <option value="5">5</option>
            <option value="7">7</option>
          </select>
        </div>
        <div class="color-control colorFormat">
          <label>Формат: </label>
          <button @click="toggleColorFormat" class="colorFormat-btn">
            <span>{{ colorFormat }}</span>
          </button>
        </div>
        <div class="color-control colorBtn">
          <button @click="generateColor" class="generateColor-btn">
            Генерировать
          </button>
        </div>
      </div>
      <div class="color__preview">
        <div class="palette-generator">
          <div v-if="palette.length > 0" class="palette-display">
            <div
              v-for="(color, index) in palette"
              :key="index"
              class="color-card"
              :style="{ backgroundColor: color.hex }"
            >
              <div class="color-info">
                <span class="color-value">
                  {{ getColorValue(color) }}
                </span>
                <button
                  @click.stop="toggleLockColor(index)"
                  class="lock-button"
                  :class="{ locked: color.locked }"
                >
                  {{ color.locked ? "🔐" : "🔓" }}
                </button>
              </div>

              <div class="click-to-copy" @click="toast(color)">Нажмите, чтобы скопировать</div>
            </div>
          </div>
        </div>
      </div>
      <div class="preview-section" :class="{ dark: darkMode }">
        <button class="theme-button" @click="toggleDarkMode">
          {{ darkMode ? "Светлый фон" : "Темный фон" }}
        </button>
        <div class="mockup">
          <div
            class="mockup-header"
            :style="{ backgroundColor: palette[0]?.hex || '#667eea' }"
          >
            <h4 :style="{ color: palette[1]?.hex || '#ffffff' }">Заголовок</h4>
          </div>

          <div
            class="mockup-body"
            :style="{ backgroundColor: palette[2]?.hex || '#ffffff' }"
          >
            <button
              class="mockup-button"
              :style="{
                backgroundColor: palette[3]?.hex || '#764ba2',
                color: palette[4]?.hex || '#ffffff',
              }"
            >
              Кнопка
            </button>

            <div
              class="mockup-card"
              :style="{
                backgroundColor: palette[5]?.hex || '#f5f5f5',
              }"
            >
              <p :style="{ color: palette[6]?.hex || '#dddddd' }">
                Текст на карточке
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import { createToast } from 'mosha-vue-toastify';
import 'mosha-vue-toastify/dist/style.css'

export default {
  name: "ColorGenerator",
  setup() {
    const palette = ref([]);
    const colorCount = ref(3);
    const darkMode = ref(false);
    const colorFormat = ref("hex");

    const hexToRgb = (hex) => {
      hex = hex.replace("#", "");
      const r = parseInt(hex.substring(0, 2), 16);
      const g = parseInt(hex.substring(2, 4), 16);
      const b = parseInt(hex.substring(4, 6), 16);
      return `rgb(${r}, ${g}, ${b})`;
    };

    const getColorValue = (color) => {
      if (colorFormat.value === "hex") {
        return color.hex;
      } else {
        return color.rgb || hexToRgb(color.hex);
      }
    };

    const toast = (color) => {
        createToast('Скопировано',
            {
              type: 'success',
              timeout: 1500,
            }
        )
        const textToCopy = getColorValue(color);
        navigator.clipboard.writeText(textToCopy)

    };

    const toggleLockColor = (index) => {
      palette.value[index].locked = !palette.value[index].locked;
      saveToLocalStorage();
    };

    const generateRandomHexColor = () => {
      const letters = "0123456789ABCDEF";
      let color = "#";
      for (let i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)];
      }
      return color;
    };

    const generateColor = () => {
      const newPalette = [];

      for (let i = 0; i < colorCount.value; i++) {
        const existingColor = palette.value[i];

        if (existingColor && existingColor.locked) {
          newPalette.push({ ...existingColor });
        } else {
          let newColor = generateRandomHexColor();

          newPalette.push({
            hex: newColor,
            rgb: hexToRgb(newColor),
            locked: false,
            copied: false,
          });
        }
      }

      palette.value = newPalette;
      saveToLocalStorage();
    };

    const saveToLocalStorage = () => {
      const dataToSave = {
        palette: palette.value,
        colorCount: colorCount.value,
        darkMode: darkMode.value,
        colorFormat: colorFormat.value,
      };
      localStorage.setItem("colorPalette", JSON.stringify(dataToSave));
    };

    const loadFromLocalStorage = () => {
      try {
        const savedData = localStorage.getItem("colorPalette");

        if (!savedData) {
          generateColor();
          return;
        }

        const parsedData = JSON.parse(savedData);
        console.log("Загружены данные:", parsedData);

        // Загружаем количество цветов
        if (parsedData.colorCount !== undefined) {
          colorCount.value = Number(parsedData.colorCount);
          console.log("Загружено количество цветов:", colorCount.value);
        }

        // Загружаем тему
        if (parsedData.darkMode !== undefined) {
          darkMode.value = Boolean(parsedData.darkMode);
          console.log("Загружена тема:", darkMode.value);
        }

        if (parsedData.colorFormat !== undefined) {
          (colorFormat.value = parsedData.colorFormat),
            console.log("Загружен формат цвета:", colorFormat.value);
        }

        // Загружаем палитру
        if (
          Array.isArray(parsedData.palette) &&
          parsedData.palette.length > 0
        ) {
          palette.value = parsedData.palette;
          console.log("Загружено цветов:", palette.value.length);
        } else {
          console.log("Палитра пуста или не массив, генерируем новую");
          generateColor();
        }
      } catch (error) {
        console.error("Ошибка загрузки из localStorage:", error);
        localStorage.removeItem("colorPalette");
        generateColor();
      }
    };
    onMounted(() => {
      loadFromLocalStorage();
    });

    const toggleDarkMode = () => {
      darkMode.value = !darkMode.value;
      saveToLocalStorage();
    };

    

    const toggleColorFormat = () => {
      if (colorFormat.value === "hex") {
        (colorFormat.value = "rgb"),
          saveToLocalStorage(),
          console.log("Загружен формат цвета:", colorFormat.value);
      } else {
        (colorFormat.value = "hex"),
          saveToLocalStorage(),
          console.log("Загружен формат цвета:", colorFormat.value);
      }
    };

    return {
      palette,
      colorCount,
      generateColor,
      getColorValue,
      toggleLockColor,
      toggleDarkMode,
      darkMode,
      toggleColorFormat,
      colorFormat,
      toast,
    };
  },
};
</script>

<style>
.page {
  display: flex;
  justify-content: center;
  flex-direction: column;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

header {
  text-align: center;
  margin-bottom: 30px;
}

header h1 {
  margin-bottom: 20px;
  color: #ffffff;
}

.generateColor-btn {
  padding: 10px 20px;
  background-color: #667eea;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
}

.generateColor-btn:hover {
  background-color: #764ba2;
}

.color__actions {
  background: white;
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 20px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.palette-generator {
  padding: 20px;
  text-align: center;
  background: white;
  border-radius: 10px;
}

.palette-display {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-top: 20px;
  flex-wrap: wrap;
}

.color-card {
  width: 180px;
  height: 200px;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 15px;
  position: relative;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
  cursor: default;
  transition: transform 0.3s ease;
}

.color-card:hover {
  transform: translateY(-5px);
}

.color-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: rgba(255, 255, 255, 0.85);
  padding: 8px 10px;
  border-radius: 5px;
}

.color-value {
  font-family: "Courier New", monospace;
  font-weight: bold;
  font-size: 0.85rem;
  color: #000;
}

.lock-button {
  background: none;
  border: none;
  font-size: 1.2rem;
  cursor: pointer;
  padding: 5px;
}

.lock-button.locked {
  color: #ffc107;
}

.click-to-copy {
  background-color: rgba(255, 255, 255, 0.85);
  padding: 8px;
  border-radius: 5px;
  cursor: pointer;
  text-align: center;
  font-size: 12px;
  transition: background-color 0.2s;
  color: black;
}

.preview-section {
  margin: 30px 0;
  padding: 25px;
  background: #f8f9fa;
  border-radius: 10px;
}

.preview-section.dark {
  background-color: #333;
  color: white;
}

.mockup {
  border: 1px solid #ccc;
  border-radius: 8px;
  margin-top: 15px;
  overflow: hidden;
}

.mockup-header {
  padding: 25px;
  text-align: center;
}

.mockup-body {
  padding: 25px;
  display: flex;
  gap: 20px;
  background: white;
}

.mockup-button {
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  font-weight: bold;
  cursor: default;
  min-width: 100px;
}

.mockup-card {
  padding: 20px;
  border-radius: 6px;
  flex: 1;
}

button {
  cursor: pointer;
}

label {
  font-weight: 600;
  color: #333;
  margin-right: 10px;
}

.theme-button {
  border-radius: 5px;
  background-color: azure;
}
</style>
