<template>
  <div class="header">
    <div style="display: flex; gap: 8px; align-items: center">
      <div class="header__title">Редактирование фото</div>
      <div style="display: flex; gap: 4px; top: 10px">
        <Button
          @click="prevHistory"
          :disabled="historyImage.length <= 1 || indexForCut == 0"
          only-icon
          variant="light-gray"
          ><Icon color="#3B82F6" name="arrow-back"
        /></Button>
        <Button
          @click="nextHistory"
          :disabled="historyImage.length <= 1 || indexForCut == historyImage.length - 1"
          style="transform: scale(-1, 1)"
          only-icon
          variant="light-gray"
          ><Icon color="#3B82F6" name="arrow-back"
        /></Button>
      </div>
    </div>
    <Icon @click="close" name="close" class="header__close" />
  </div>
  <div style="display: flex; flex-direction: column; gap: 24px">
    <div class="header_btns">
      <Button
        @click="changeCatOptions(0, 'Настройки цвета')"
        :class="{ active: activeBtn == 0 }"
        class="default"
        variant="light"
      >
        <Icon name="edit-pencil" color="#3B82F6" />
        Настройки цвета
      </Button>
      <Button
        @click="changeCatOptions(1, 'Поворот')"
        :class="{ active: activeBtn == 1 }"
        class="default"
        variant="light"
      >
        <Icon name="reload" color="#3B82F6" style="transform: scale(-1, 1)" />
        Поворот
      </Button>
      <Button
        @click="changeCatOptions(2, 'Обрезка')"
        :class="{ active: activeBtn == 2 }"
        class="default"
        variant="light"
      >
        <Icon name="minimize" color="#3B82F6" />
        Обрезка
      </Button>
      <Button
        @click="changeCatOptions(3, 'Отражение')"
        :class="{ active: activeBtn == 3 }"
        class="default"
        variant="light"
      >
        <Icon name="flip" color="#3B82F6" />
        Отражение
      </Button>
      <Button
        @click="changeCatOptions(4, 'История')"
        :class="{ active: activeBtn == 4 }"
        class="default"
        variant="light"
      >
        <Icon name="burger" color="#3B82F6" />
        История
      </Button>
    </div>
    <div class="editor__wrapper">
      <div ref="stageWrapper" class="stage__wrapper">
        <Loader v-if="isLoading" />
        <v-stage v-show="!isLoading" ref="stageRef" :config="configStage">
          <v-layer ref="layerRef">
            <v-image ref="imageNode" :config="imageConfig" />
            <v-rect ref="rectRef" :config="rectCrop" v-if="selected" />
            <v-transformer ref="tranRef" :config="tranConfig" v-if="selected" />
          </v-layer>
        </v-stage>
      </div>
      <div class="controls__wrapper">
        <div class="wrapper" v-if="activeBtn == 0">
          <label class="input__wrapper">
            Яркость
            <input
              @input="
                applyBrightness(parseFloat(($event.target as HTMLInputElement).value)),
                  parseFloat(($event.target as HTMLInputElement).value) != 0 ? (dirty = true) : null
              "
              type="range"
              min="-1"
              max="1"
              step="0.01"
              value="0"
              ref="brightnessInput"
            />
          </label>
          <label class="input__wrapper">
            Контрасть
            <input
              @input="
                applyContrast(parseFloat(($event.target as HTMLInputElement).value)),
                  parseFloat(($event.target as HTMLInputElement).value) != 0 ? (dirty = true) : null
              "
              type="range"
              min="-100"
              max="100"
              step="1"
              value="0"
              ref="contrastInput"
            />
          </label>
          <label class="input__wrapper">
            Насыщенность
            <input
              @input="
                applySaturation(parseFloat(($event.target as HTMLInputElement).value)),
                  parseFloat(($event.target as HTMLInputElement).value) != 0 ? (dirty = true) : null
              "
              type="range"
              min="-2"
              max="2"
              step="0.1"
              value="0"
              ref="saturationInput"
            />
          </label>
        </div>
        <div class="wrapper" v-if="activeBtn == 1">
          <Button
            class="default"
            @click="(dirty = true), rotate(90)"
            variant="light-outline"
            style="color: #282b2e"
          >
            <Icon color="#3B82F6" name="reload" style="transform: scale(-1, 1)" />
            Повернуть вправо на 90°
          </Button>
          <Button
            class="default"
            @click="(dirty = true), rotate(-90)"
            variant="light-outline"
            style="color: #282b2e"
          >
            <Icon color="#3B82F6" name="reload" />
            Повернуть влево на 90°
          </Button>
        </div>
        <div class="wrapper" v-if="activeBtn == 2">
          <Button
            class="default"
            @click="selectImage(), (dirty = true)"
            variant="light-outline"
            style="color: #282b2e"
          >
            <Icon color="#3B82F6" name="minimize" />
            Кадрировать
          </Button>
          <span class="w-full text-center text-red">В разработке</span>
        </div>
        <div class="wrapper" v-if="activeBtn == 3">
          <Button
            class="default"
            @click="flipHorizontal(), (dirty = true)"
            variant="light-outline"
            style="color: #282b2e"
          >
            <Icon color="#3B82F6" name="flip" />
            Отразить по горизонтали
          </Button>
          <Button
            class="default"
            @click="flipVertical(), (dirty = true)"
            variant="light-outline"
            style="color: #282b2e"
          >
            <Icon color="#3B82F6" name="flip" />
            Отразить по вертикали
          </Button>
        </div>
        <div class="wrapper" v-if="activeBtn == 4">
          <span class="history__header">История</span>
          <div
            style="display: flex; flex-direction: column; gap: 6px"
            v-for="(item, index) in historyImage"
            :key="item.date"
          >
            <div style="display: flex; gap: 4px">
              <Icon color="#D7D7D7" name="clock" />
              <span>{{
                `${new Date(item.date).getHours()}:${
                  String(new Date(item.date).getMinutes()).length == 1
                    ? '0' + new Date(item.date).getMinutes()
                    : new Date(item.date).getMinutes()
                }`
              }}</span>
            </div>
            <span>{{ item.title }}</span>
            <span
              style="
                color: #3b82f6;
                border-bottom: 1px dashed #3b82f6;
                width: max-content;
                cursor: pointer;
              "
              @click="loadImage(item.src), (indexForCut = index)"
            >
              Вернуть
            </span>
          </div>
        </div>
      </div>
    </div>
    <div class="footer_btns">
      <Button @click="saveChanges(), $emit('saveImage', { file: imgFile, image: imageObj })">
        Сохранить
      </Button>
      <Button @click="$emit('close')" variant="light-gray" style="color: #3b82f6">Отмена</Button>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { onMounted, onUnmounted, ref, watchEffect } from 'vue';
import { useDebounceFn, useThrottleFn } from '@vueuse/core';
import Konva from 'konva';

import Button from '../button/Button.vue';
import Loader from '../Loader.vue';

import Icon from '~/components/icon/Icon.vue';

interface IEditorProps {
  defImg: string;
  imgFile: any;
}

const props = defineProps<IEditorProps>();

const emit = defineEmits<{
  (event: 'saveImage', data: { file: any; image: any }): void;
  (event: 'close'): void;
}>();

const configStage = ref({ width: 400, height: 400 });
const imageConfig = ref({
  x: 0,
  y: 0,
  image: new Image(),
  width: 0,
  height: 0,
  rotation: 0,
  offsetX: 0,
  offsetY: 0,
  filters: [Konva.Filters.Brighten, Konva.Filters.Contrast],
  brightness: 0,
  contrast: 0,
  saturation: 0,
});

var rectCrop = ref({
  x: 0,
  y: 0,
  width: 0,
  height: 0,
  offsetX: 0,
  offsetY: 0,
  fill: 'transparent',
  stroke: 'black',
  strokeWidth: 1,
  strokeScaleEnabled: true,
  draggable: true,
});

const isLoading = ref(false);
const imageObj = ref<HTMLImageElement | null>(null);
const stageRef = ref();
const layerRef = ref();
const imageNode = ref();
const tranRef = ref();
const rectRef = ref();
const saturationInput = ref();
const contrastInput = ref();
const brightnessInput = ref();
const selected = ref(false);
const dirty = ref(false);
const activeBtn = ref(0);
const historyImage = ref<any>([]);
const title = ref('Настройки цвета');
const indexForCut = ref<number | null>(0);
const stageWrapper = ref<HTMLDivElement>();
let tranConfig = ref({
  nodes: [] as Konva.Node[],
  centeredScaling: true,
  rotateEnabled: false,
  keepRatio: false,
});

const close = () => {
  emit('close');
};

const scale = () => {
  if (imageObj.value) {
    if (stageWrapper.value) {
      if (
        stageWrapper.value.clientHeight > imageObj.value.height &&
        stageWrapper.value.clientWidth > imageObj.value.width
      ) {
        imageNode.value.getNode().attrs.scaleX = 1;
        imageNode.value.getNode().attrs.scaleY = 1;
      } else {
        if (
          imageObj.value.width > imageObj.value.height &&
          imageObj.value.height * (stageWrapper.value.clientWidth / imageObj.value.width) <
            stageWrapper.value.clientHeight
        ) {
          imageNode.value.getNode().attrs.scaleX =
            stageWrapper.value.clientWidth / imageObj.value.width;
          imageNode.value.getNode().attrs.scaleY =
            stageWrapper.value.clientWidth / imageObj.value.width;
        } else {
          imageNode.value.getNode().attrs.scaleX =
            stageWrapper.value.clientHeight / imageObj.value.height;
          imageNode.value.getNode().attrs.scaleY =
            stageWrapper.value.clientHeight / imageObj.value.height;
        }
      }
    }
  }
  imageNode.value.getNode().clearCache();
};

function loadImageAsync(src: string) {
  isLoading.value = true;
  return new Promise((resolve) => {
    imageObj.value = new Image();
    imageObj.value.src = src;
    imageObj.value.onload = resolve;
  });
}

const loadImage = async (src: string) => {
  await loadImageAsync(src).then(() => {
    imageObj.value ? (imageConfig.value.image = imageObj.value) : null;
    if (imageObj.value) {
      if (stageWrapper.value) {
        imageConfig.value.width = imageObj.value.width;
        imageConfig.value.height = imageObj.value.height;

        configStage.value.width = stageWrapper.value.clientWidth;
        configStage.value.height = stageWrapper.value.clientHeight;

        imageConfig.value.x = configStage.value.width / 2;
        imageConfig.value.y = configStage.value.height / 2;

        imageConfig.value.rotation = 0;

        imageConfig.value.offsetX = imageConfig.value.width / 2;
        imageConfig.value.offsetY = imageConfig.value.height / 2;
      }
    }
  });
  isLoading.value = false;
  // scale();
};

const addHistory = (title: string, src: string) => {
  historyImage.value.push({
    title: title,
    src: src,
    date: Number(new Date()),
  });
  indexForCut.value = historyImage.value.length - 1;
};

const prevHistory = () => {
  if (indexForCut.value != null && indexForCut.value > 0) {
    loadImage(historyImage.value[indexForCut.value - 1].src);
    indexForCut.value -= 1;
  }
};

const nextHistory = () => {
  if (indexForCut.value != null && indexForCut.value < historyImage.value.length) {
    loadImage(historyImage.value[indexForCut.value + 1].src);
    indexForCut.value += 1;
  }
};

const deleteUnactualHistory = () => {
  if (
    (activeBtn.value == 4 || dirty.value == true) &&
    historyImage.value.length > 1 &&
    indexForCut.value != null
  ) {
    historyImage.value.splice(indexForCut.value + 1);
  }
};

const changeCatOptions = (cat: number, str: string) => {
  deleteUnactualHistory();
  activeBtn.value = cat;
  dirty.value ? saveChanges() : null;
  dirty.value = false;
  title.value = str;
};

onMounted(async () => {
  loadImage(props.defImg);
  addHistory('Оригинал', imageObj.value?.src as string);
  scale();
});

watchEffect(() => {
  rectRef.value ? (tranConfig.value.nodes = [rectRef.value.getNode()]) : null;
});

function selectImage() {
  rectCrop.value.width = configStage.value.width;
  rectCrop.value.height = configStage.value.height;
  // rectCrop.value.width = imageConfig.value.width * imageNode.value.getNode().attrs.scaleX;
  // rectCrop.value.height = imageConfig.value.height * imageNode.value.getNode().attrs.scaleY;

  // rectCrop.value.x = imageConfig.value.width / 2;
  // rectCrop.value.y = imageConfig.value.height / 2;

  // rectCrop.value.offsetX = imageConfig.value.width / 2;
  // rectCrop.value.offsetY = imageConfig.value.height / 2;

  selected.value = true;
}

const flipHorizontal = useThrottleFn(() => {
  imageNode.value.getNode().to({
    scaleX: -imageNode.value.getNode().scaleX(),
  });
}, 1000);
const flipVertical = useThrottleFn(() => {
  imageNode.value.getNode().to({
    scaleY: -imageNode.value.getNode().scaleY(),
  });
  console.log(imageNode.value.getNode().attrs.scaleY);
}, 1000);

const rotate = (num: number) => {
  imageConfig.value.rotation += num;
  if (imageConfig.value.rotation >= 360) {
    imageConfig.value.rotation = 0;
  }
  if (imageObj.value) {
    const oldWidth = imageObj.value.width;
    imageObj.value.width = imageObj.value?.height;
    imageObj.value.height = oldWidth;
    scale();
  }
};

const applyBrightness = useDebounceFn((value: number) => {
  imageNode.value.getNode().attrs.brightness = value;
  applyFilters();
}, 100);
const applyContrast = useDebounceFn((value: number) => {
  imageNode.value.getNode().attrs.contrast = value;
  applyFilters();
}, 100);
const applySaturation = useDebounceFn((value: number) => {
  imageNode.value.getNode().attrs.saturation = value;
  applyFilters();
}, 100);
function applyFilters() {
  imageNode.value
    .getNode()
    .filters([Konva.Filters.Brighten, Konva.Filters.Contrast, Konva.Filters.HSL]);
  imageNode.value.getNode().cache();
  imageNode.value.getNode().getLayer().batchDraw();
}
async function saveChanges() {
  imageNode.value.getNode().attrs.scaleX < 0
    ? (imageNode.value.getNode().attrs.scaleX = -1)
    : (imageNode.value.getNode().attrs.scaleX = 1);
  imageNode.value.getNode().attrs.scaleY < 0
    ? (imageNode.value.getNode().attrs.scaleY = -1)
    : (imageNode.value.getNode().attrs.scaleY = 1);
  imageNode.value.getNode().clearCache();

  const image = imageNode.value.getNode();
  let changedImg;
  if (tranRef.value) {
    const transformer = tranRef.value.getNode();
    const crop = {
      x: Math.round(transformer.x() - image.x() + imageConfig.value.offsetX + 10),
      y: Math.round(transformer.y() - image.y() + imageConfig.value.offsetY + 10),
      width: Math.round(transformer.width() * transformer.scaleX()),
      height: Math.round(transformer.height() * transformer.scaleY()),
      mimeType: 'image/jpeg',
    };
    changedImg = image.toDataURL(crop);
  } else {
    changedImg = image.toDataURL({ mimeType: 'image/jpeg' });
  }
  selected.value = false;

  await loadImage(changedImg);
  addHistory(title.value, imageObj.value?.src as string);

  saturationInput.value ? (saturationInput.value.value = 0) : null;
  contrastInput.value ? (contrastInput.value.value = 0) : null;
  brightnessInput.value ? (brightnessInput.value.value = 0) : null;

  imageNode.value.getNode().getLayer().batchDraw();
  scale();
}

onMounted(() => {
  window.addEventListener('resize', scale);
});
onUnmounted(() => {
  window.removeEventListener('resize', scale);
});
</script>

<style lang="scss" scoped>
.default {
  border: 1px solid #e9e9e9;
}
.header_btns {
  display: flex;
  gap: 8px;
  .active {
    background-color: #f4f6f9;
    border: 1px solid transparent;
  }
}
.editor__wrapper {
  display: flex;
  gap: 24px;
  max-height: 60vh;
  .stage__wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
    // min-width: 500px;
    // min-height: 500px;
    width: 80vw;
    height: 60vh;
    border: 1px solid #e9e9e9;
    border-radius: $big-br;
  }
  .controls__wrapper {
    display: flex;
    flex-direction: column;
    overflow: auto;
    width: 100%;
    max-width: 200px;
    .wrapper {
      display: flex;
      flex-direction: column;
      gap: 16px;
      margin-bottom: 32px;
      .history__header {
        margin: 0;
        @extend %text-lg;
        font-weight: 700;
        margin-bottom: 32px;
      }
    }
  }
}
.footer_btns {
  display: flex;
  gap: 8px;
}

.input__wrapper {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  @extend %text-sm;
  color: $dark;
}
</style>
