<template>
  <div class="box-wrapper">
    <div
      class="box"
      :style="{ top: bTop + '%', left: bLeft + '%', width: bWidth + '%', height: bHeight + '%' }"
      :class="{ 'active': bActive, 'dragActive': bDrag, 'resizeActive': bResize }"
      @mousedown="$emit('onSelect', bIndex)"
      @touchstart="$emit('onSelect', bIndex)"
    >
      <div
        class="flex flex-center item-center"
        style="border: 1px solid black; height: 100%"
      >
        <a
          v-if="bActive"
          @mousedown="enableDrag"
          @touchstart="enableDrag"
        >
          <q-icon
            name="add"
            class="text-danger text-weight-bold"
            :class="{ 'cursor-grab': !bDrag, 'cursor-active': bDrag }"
          />
        </a>
      </div>

      <!-- REMOVE BUTTON -->
      <a
        v-if="bActive"
        class="box-delete"
        @mousedown="removeMyself"
        @touchstart.stop="removeMyself"
      >
        <q-icon name="cancel" />
      </a>
      <!-- / - REMOVE BUTTON -->

      <!-- RESIZE - BOTTOM - LEFT -->
      <a
        v-if="bActive"
        class="box-resize resize-bottom-left"
        @mousedown="resizeClick('bottom-left')"
        @touchstart="resizeClick('bottom-left')"
      >
        <q-icon name="open_in_full" />
      </a>
      <!-- RESIZE - TOP LEFT -->
      <a
        v-if="bActive"
        class="box-resize resize-top-left"
        @mousedown="resizeClick('top-left')"
        @touchstart="resizeClick('top-left')"
      >
        <q-icon name="open_in_full" />
      </a>
      <!-- RESIZE - BOTTOM - RIGHT -->
      <a
        v-if="bActive"
        class="box-resize resize-bottom-right"
        @mousedown="resizeClick('bottom-right')"
        @touchstart="resizeClick('bottom-right')"
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
const emit = defineEmits(['onSelect', 'onDelete', 'onDrag', 'onResize'])
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
  bDrag: {
    type: Boolean,
    required: true
  },
  bResize: {
    type: Boolean,
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

function resizeClick(resizeOf) {
  emit('onResize', resizeOf)
}

function enableDrag() {
  emit('onDrag')
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
  &.dragActive {
    cursor: grabbing !important;
    background-color: rgba(0, 244, 0, 0.2);
  }
  &.resizeActive {
    background-color: rgba(0, 244, 0, 0.2);
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
  font-size: 1rem;
  color: white;
}

.resize-bottom-right {
  bottom: -14px;
  right: -10px;
  transform: rotate(90deg);
  cursor: se-resize;
}

.resize-bottom-left {
  bottom: -14px;
  left: -10px;
  cursor: sw-resize;
}

.resize-top-left {
  top: -14px;
  left: -10px;
  transform: rotate(90deg);
  cursor: nw-resize;
}
.cursor-grab {
  cursor: grab;
}

.cursor-active {
  border: 1px solid black;
  border-radius: 100%;
}
</style>
