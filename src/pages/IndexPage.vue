<template>
  <div class="container">
    <div
      id="image-wrapper"
      ref="wrapper"
      @mousedown="startDrawingBox"
      @mousemove="changeBox"
      @mouseup="stopDrawingBox"
    >
      <img src="../assets/caterpillar.jpg">
      <RectangularBox
        v-if="drawingBox.active"
        :b-width="drawingBox.width"
        :b-height="drawingBox.height"
        :b-top="drawingBox.top"
        :b-left="drawingBox.left"
        :b-active="drawingBox.active"
        @udpate-resize="updateResize"
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
        @on-select="makeBoxActive"
        @on-delete="removeBox"
        @change-box="changeBox"
        @update-resize="updateResize"
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

const getCoursorLeft = (e, base, offset) => {
  // 10 in px is distance from left of the browser window
  return ((e.pageX - offset) / base) * 100; // converting the distance to % for responsiveness
};

const getCoursorTop = (e, base, offset) => {
  // 10 in px is distance from top of the browser
  return ((e.pageY - offset) / base) * 100; // converting the distance to % for responsiveness
};

const resize = ref(false);
function updateResize(val) {
  resize.value = val;
}

const activeBoxIndex = ref(0);

function makeBoxActive(i) {
  activeBoxIndex.value = i;
  resize.value = true;
}

const drawingBox = reactive({
  active: false,
  top: 0,
  left: 0,
  height: 0,
  width: 0,
  label: ''
});
const boxes = ref([]);

function startDrawingBox(e) {
  e.preventDefault()
  // determine if resizing by activeBox value
  const activeBox = boxes.value[activeBoxIndex.value]
  console.log(activeBox);
  if (resize.value) {
    console.log('resize?')
    const activeBox = boxes.value[activeBoxIndex.value]
    drawingBox.width = activeBox.width
    drawingBox.height = activeBox.height
    drawingBox.top = activeBox.top
    drawingBox.left = activeBox.left
    drawingBox.active = true
  } else {
    console.log('not resize?')
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
  if (drawingBox.active) {
    const offsetLeft = wrapper.value.offsetLeft
    const offsetTop = wrapper.value.offsetTop
    const width = getCoursorLeft(e, wrapper.value.clientWidth, offsetLeft) - drawingBox.left;
    const height = getCoursorTop(e, wrapper.value.clientHeight, offsetTop) - drawingBox.top;
    drawingBox.width = width;
    drawingBox.height = height;
  }
}

function stopDrawingBox() {
  if (drawingBox.active) {
    if (drawingBox.width > 4) {
      const obj = { ...drawingBox }
      if (resize.value) {
        boxes.value[activeBoxIndex.value].height = obj.height;
        boxes.value[activeBoxIndex.value].width = obj.width;
      } else {
        boxes.value.push(obj);
        activeBoxIndex.value = boxes.value.length - 1;
      }
    }
    drawingBox.width = 0
    drawingBox.height = 0
    drawingBox.top = 0
    drawingBox.left = 0
    drawingBox.active = false

    resize.value = false

  }
}

function removeBox(i) {
  boxes.value = boxes.value.filter((_, index) => {
    return index !== i;
  });
  activeBoxIndex.value = 0;
  resize.value = false;
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
