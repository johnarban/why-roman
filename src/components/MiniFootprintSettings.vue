<template>
  <div
    class="mini-fp-settings pa-2"
    :style="{ '--fp-color': color }"
  >
    <!-- align-center so the label and checkbox sit centred against a taller
         #action button rather than riding along its top edge -->
    <div class="d-flex flex-wrap justify-space-between align-center">
      <div
        v-if="showGo"
        class="go-option"
      >
        <button
          type="button"
          class="go-button"
          :aria-label="`Point to ${label}`"
          @click="emit('go')"
        >
          <v-icon
            icon="mdi-crosshairs-gps"
            size="small"
          />
        </button>
      </div>
      <div class="show-option">
        <label>
          <input
            v-model="shown"
            type="checkbox"
          />
          <span class="ml-2"> {{ label }} </span>
        </label>
      </div>
      <div
        v-if="showFill && showColor"
        class="color-option"
      >
        <label>
          <!-- <span class="mr-2">Fill</span> -->
          <input
            v-model="color"
            type="color"
          />
        </label>
      </div>
      <div
        v-if="showFill"
        class="fill-option"
        :class="{ 'fill-option-solo': !showColor }"
      >
        <label>
          <span class="mr-2">Fill</span>
          <input
            v-model="fill"
            type="checkbox"
          />
        </label>
      </div>
      <div v-if="showOpacity">
        <input
          v-model.number="opacity"
          type="range"
          min="0"
          max="1"
          step="0.01"
        />
      </div>
      <!-- for an action belonging to this layer, e.g. "zoom to pixel scale" -->
      <slot name="action"></slot>
    </div>
  </div>
</template>

<script setup lang="ts">
import { watch } from "vue";

interface Props {
  /** footprint label, e.g. "Roman" */
  label: string;
  /** offer the fill toggle. most footprints aren't filled */
  showFill?: boolean;
  /** the colour picker, alongside the fill toggle. off moves fill into its spot */
  showColor?: boolean;
  /** the per-footprint opacity slider. off where the box is already busy */
  showOpacity?: boolean;
  /** a crosshairs button that asks the parent to point the view at this footprint */
  showGo?: boolean;
}

withDefaults(defineProps<Props>(), {
  showFill: false,
  showColor: true,
  showOpacity: true,
  showGo: false,
});

const emit = defineEmits<{
  show: [show: boolean],
  go: [],
}>();

const opacity = defineModel<number>("opacity", { required: true });
const fill = defineModel<boolean>("fill", { required: true });
const color = defineModel<string>("color", { required: true });

// the footprint's own opacity-based show/hide. `show` belongs to the tour,
// which flips footprints per step, so this control can't use it
const shown = defineModel<boolean>("visible", { required: true });
watch(shown, (value: boolean) => emit("show", value));
</script>

<style scoped>
.mini-fp-settings {
  border: 1px solid var(--fp-color);
  border-radius: 4px;
  background: black;
  color: var(--fp-color);
  padding: 0.5em;
  pointer-events: auto;
  user-select: none;
}

/* The slider sits under the label rather than beside it. */
.mini-fp-settings > div > div:last-child {
  flex-basis: 100%;
}

.show-option label {
  display: flex;
  align-items: center;
}

.show-option {
  margin-right: auto;
}

/* so a long label wraps beside the box rather than squashing it */
.show-option input[type="checkbox"] {
  flex: 0 0 auto;
}

.go-button {
  background: none;
  border: none;
  padding: 0;
  margin-right: 0.5em;
  color: var(--fp-color);
  cursor: pointer;
  display: flex;
  align-items: center;
}

.fill-option {
  padding-left: 0.5em;
}

/* sits on the label's row; the override beats the last-child rule above */
.fill-option-solo {
  padding-left: 0;
}

.mini-fp-settings .fill-option-solo:last-child {
  flex-basis: auto;
}

label > span {
  color: white;
}
input {
  /* this color will only get applied for certain modern browsers, but is a nice touch */
  accent-color: var(--fp-color);
  cursor: pointer;
}

input[type="range"] {
  width: 100%;
}
</style>
