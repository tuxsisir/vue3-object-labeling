<template>
  <div class="box-wrapper">
    <div
      class="box"
      :style="{ top: bTop + '%', left: bLeft + '%', width: bWidth + '%', height: bHeight + '%' }"
      :class="{ 'active': bActive }"
      @mousedown="$emit('onSelect', bIndex)"
    >
      <!-- REMOVE BUTTON -->
      <a
        v-if="bActive"
        class="box-delete"
        @click="removeMyself"
      >
        <q-icon name="cancel" />
      </a>
      <!-- / - REMOVE BUTTON -->

      <!-- RESIZE -->
      <a
        v-if="bActive"
        class="box-resize"
        @click="resizeClick"
      >
        <q-icon name="open_in_full" />
      </a>
      <!-- / - RESIZE -->
      <div class="text-white text-weight-bold">
        {{ bLabel }}
      </div>
      <!--

      <div>
        Active: {{ bActive }}
      </div>
      <div>
        Btop: {{ bTop }}
      </div>
      <div>
        BLeft: {{ bLeft }}
      </div>
      <div>
        BWidth: {{ bWidth }}
      </div>
      <div>
        BHeight: {{ bHeight }}
      </div>
      -->
    </div>
  </div>
</template>

<script setup>
const emit = defineEmits(['onSelect', 'onDelete', 'changeBox', 'updateResize'])
const props = defineProps({
  bTop: {
    type: Number,
    required: true
  },
  bLeft: {
    type: Number,
    required: true
  },
  bWidth: {
    type: Number,
    required: true
  },
  bHeight: {
    type: Number,
    required: true
  },
  bActive: {
    type: Boolean,
    required: false,
    default() {
      return false
    }
  },
  bLabel: {
    type: String,
    required: false,
    default() {
      return ''
    }
  },
  bIndex: {
    type: Number,
    required: true,
    required: false,
    default() {
      return 0
    }
  }
})
function removeMyself() {
  emit('onDelete', props.bIndex)
}
// function onMouseMove(event) {
//   emit('changeBox', event)
// }
function resizeClick() {
  emit('updateResize', true)
}
</script>

<style lang="scss" scoped>
.box {
  position: absolute;
  border: 1px #90ee90 dashed;

  &:hover,
  &.active {
    background-color: rgba(0, 244, 0, 0.8);
  }

  z-index: 3;
}

.box-delete {
  position: absolute;
  z-index: 6;
  font-weight: bold;
  color: red;
  cursor: pointer;
}

.box-delete {
  top: -17px;
  right: -14px;
  font-size: 1.3rem;
}

.box-resize {
  z-index: 6;
  position: absolute;
  cursor: nwse-resize;
  bottom: -14px;
  right: -10px;
  transform: rotate(90deg);
  font-size: 1.3rem;
  color: white;
}
</style>
