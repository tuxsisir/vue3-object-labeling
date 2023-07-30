<template>
  <div class="container">
    <div
      id="image-wrapper"
      ref="wrapper"
      @mousedown="startDrawingBox"
      @mousemove="changeBox"
      @mouseup="stopDrawingBox"
      @touchstart="startDrawingBox"
      @touchmove="changeBox"
      @touchend="stopDrawingBox"
    >
      <q-img src="../assets/caterpillar.jpg" />
      <RectangularBox
        v-if="drawingBox.active"
        :b-width="drawingBox.width"
        :b-height="drawingBox.height"
        :b-top="drawingBox.top"
        :b-left="drawingBox.left"
        :b-active="drawingBox.active"
        :b-drag="drag"
        :b-resize="resize"
      />
      <RectangularBox
        v-for="(box, i) in boxes"
        :key="i"
        :b-top="box.top"
        :b-left="box.left"
        :b-width="box.width"
        :b-height="box.height"
        :b-label="box.label"
        :b-active="i === activeBoxIndex"
        :b-index="i"
        :b-drag="drag"
        :b-resize="resize"
        @on-drag="drag = true"
        @on-resize="enableResize"
        @on-select="makeBoxActive"
        @on-delete="removeBox"
      />
    </div>
    <!-- CONTROLS -->
    <div class="row">
      <div class="col col-sm-12 col-xs-12">
        <h6 class="q-ma-sm">
          Labels
        </h6>
        <q-list
          v-if="boxes.length > 0"
          bordered
          separator
          class="q-ma-sm"
        >
          <q-item
            clickable
            v-ripple
            v-for="(box, i) in boxes"
            :key="i"
            :active="i === activeBoxIndex"
            active-class="bg-teal-1 text-grey-8"
          >
            <q-item-section>
              <q-input
                outlined
                v-model="box.label"
                label="Write name for the box!"
                dense
                @focus="activateBoxFromLabel(i)"
              >
                <template #append>
                  <q-icon
                    name="close"
                    @click="removeBox(i)"
                    class="cursor-pointer"
                  />
                </template>
              </q-input>
            </q-item-section>
          </q-item>
        </q-list>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref, reactive } from "vue";
import RectangularBox from '../components/RectangularBox.vue'

const wrapper = ref(null);
const drag = ref(false);

const getCoursorLeft = (e, base, offset) => {
  return ((e.pageX - offset) / base) * 100; // converting the distance to % for responsiveness
};

const getCoursorTop = (e, base, offset) => {
  return ((e.pageY - offset) / base) * 100; // converting the distance to % for responsiveness
};

const resize = ref(false);
const resizeType = ref(null);

function enableResize(resizeOf) {
  resize.value = true;
  resizeType.value = resizeOf
}

const activeBoxIndex = ref(0);

function makeBoxActive(i) {
  activeBoxIndex.value = i;
}

const drawingBox = reactive({
  active: false,
  top: 0,
  left: 0,
  height: 0,
  width: 0,
  label: 'Untitled Box'
});
const boxes = ref([]);

function startDrawingBox(e) {
  e.preventDefault()
  // determine if resizing by activeBox value
  if (drag.value) {
    console.log('dragging start')
  } else if (resize.value) {
    console.log('resizing start')
  } else {
    const offsetLeft = wrapper.value.offsetLeft;
    const offsetTop = wrapper.value.offsetTop;
    drawingBox.width = 0
    drawingBox.height = 0
    drawingBox.top = getCoursorTop(e, wrapper.value.clientHeight, offsetTop)
    drawingBox.left = getCoursorLeft(e, wrapper.value.clientWidth, offsetLeft)
    drawingBox.active = true
  }
}
function changeBox(e) {
  // while dragging the existing box
  if (drag.value) {
    // we will drag the value here, let's change the existing values:
    // top
    // left
    // only to move the box around
    const offsetLeft = wrapper.value.offsetLeft;
    const offsetTop = wrapper.value.offsetTop;
    const newTop = getCoursorTop(e, wrapper.value.clientHeight, offsetTop);
    const newLeft = getCoursorLeft(e, wrapper.value.clientWidth, offsetLeft);
    const dragTop = newTop - (boxes.value[activeBoxIndex.value].height / 2);
    const dragLeft = newLeft - (boxes.value[activeBoxIndex.value].width / 2);

    const currentHeight = boxes.value[activeBoxIndex.value].height;
    const currentWidth = boxes.value[activeBoxIndex.value].width;

    // add limits on dragging so that it won't go beyond the desired div
    boxes.value[activeBoxIndex.value].top = (currentHeight + dragTop) <= 100 ? dragTop : 100 - currentHeight;
    boxes.value[activeBoxIndex.value].left = (currentWidth + dragLeft) <= 100 ? dragLeft: 100 - currentWidth; // this sets the cursor to midpoint

  }

  // while resizing the existing box
  if (resize.value) {
    console.log('start resizing ... ');
    const offsetTop = wrapper.value.offsetTop;
    const offsetLeft = wrapper.value.offsetLeft;
    const newHeight = getCoursorTop(e, wrapper.value.clientHeight, offsetTop);
    const newWidth = getCoursorLeft(e, wrapper.value.clientWidth, offsetLeft);

    if (resizeType.value === "top-left") {

      // on resizing from top right, increase top and left values
      const heightDiff = newHeight - boxes.value[activeBoxIndex.value].top;
      const widthDiff = newWidth - boxes.value[activeBoxIndex.value].left;
      boxes.value[activeBoxIndex.value].top = newHeight;
      boxes.value[activeBoxIndex.value].left = newWidth;

      // also need to increase both height and width
      boxes.value[activeBoxIndex.value].height -= heightDiff;
      boxes.value[activeBoxIndex.value].width -= widthDiff;

    } else if (resizeType.value === "bottom-right") {
      // on resizing from bottom, change the height and width values
      const resizedHeight = newHeight - boxes.value[activeBoxIndex.value].top
      const resizedWidth = newWidth - boxes.value[activeBoxIndex.value].left;

      const currentTop = boxes.value[activeBoxIndex.value].top;
      const currentLeft = boxes.value[activeBoxIndex.value].left;

      boxes.value[activeBoxIndex.value].height = (resizedHeight + currentTop) <= 100 ? resizedHeight : 100 - currentTop;
      boxes.value[activeBoxIndex.value].width = (resizedWidth + currentLeft) <= 100 ? resizedWidth: 100 - currentLeft;

    } else if (resizeType.value === "bottom-left") {

      // Calculate the height and width differences
      // const heightDiff = newHeight - boxes.value[activeBoxIndex.value].top;
      const widthDiff = newWidth - boxes.value[activeBoxIndex.value].left;

      const currentTop = boxes.value[activeBoxIndex.value].top;

      const resizedHeight = newHeight - boxes.value[activeBoxIndex.value].top;

      // Update the height and width of the box
      boxes.value[activeBoxIndex.value].height = (resizedHeight + currentTop) <= 100 ? resizedHeight : 100 - currentTop;

      boxes.value[activeBoxIndex.value].width -= widthDiff;

      // Update the top and left positions of the box
      boxes.value[activeBoxIndex.value].left = newWidth;
    }
  }

  // while drawing new box
  if (drawingBox.active) {
    const offsetLeft = wrapper.value.offsetLeft
    const offsetTop = wrapper.value.offsetTop

    const width = getCoursorLeft(e, wrapper.value.clientWidth, offsetLeft) - drawingBox.left;
    const height = getCoursorTop(e, wrapper.value.clientHeight, offsetTop) - drawingBox.top;

    drawingBox.width = width;
    drawingBox.height = (drawingBox.top + height) <= 100 ? height: 100 - drawingBox.top;
  }
}

function stopDrawingBox() {
  if (drawingBox.active) {
    if (drawingBox.width > 4) {
      const obj = { ...drawingBox }
      boxes.value.push(obj);
      activeBoxIndex.value = boxes.value.length - 1;
    }
    drawingBox.width = 0
    drawingBox.height = 0
    drawingBox.top = 0
    drawingBox.left = 0
    drawingBox.active = false

    resize.value = false
    drag.value = false
  } else {
    // for drag and resize active
    drag.value = false
    resize.value = false
  }
}

function removeBox(i) {
  boxes.value = boxes.value.filter((_, index) => {
    return index !== i;
  });
  activeBoxIndex.value = 0;
  drawingBox.active = false;
}
function activateBoxFromLabel(i) {
  activeBoxIndex.value = i;
}
</script>
<style lang="scss">
#image-wrapper {
  width: 100%;
  background-repeat: no-repeat;
  position: relative;
  cursor: cell;
}

img {
  width: 100%;
}
</style>
