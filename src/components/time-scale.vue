<template>
  <div
    @mouseup="mouseLeftUp()"
    style="height: 20px; padding: 10px; padding-right: 30px"
  >
    <div class="day">
      <div class="scale">
        <div class="unit" v-for="(i, index) in _range()" :key="index"
          @mousedown="mouseLeftDown(i)"
          @mouseover="mouseMove(i)"
          :style="{background: scaleColors[initValue[indexLine].value[i]],width: scaleSegmentWidth,}"
        >
          <div
            class="one"
            :class="{ lineR: (i + 1) % 4 == 0, lineL: i == 0 }"
          ></div>
          <div
            class="two"
            :class="{ lineR: (i + 1) % 2 == 0, lineL: i == 0 }"
          ></div>
          <div class="three" :class="{ lineL: i == 0 }"></div>
          <div
            class="three"
            :class="{ lineL: i == 1 }"
            :style="{ background: _eventBackground(i) }"
          ></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      mouseLeftPressed: false,
    };
  },
  props: {
    indexLine: 0,
    zoomFactor: { type: Number },
    currTime: { type: Number },
    currSwitchstate: 0,
    eventTrigger: {
      type: Boolean,
      default: true,
    },
    scaleColors: {
      type: Array,
      default: function () {
        return [];
      },
    },
    initValue: {
      type: Object,
      default: function () {
        return {};
      },
    },
  },
  computed: {
    scaleSegmentWidth: function () {
      return "calc(100% / 96 * " + this.zoomFactor + ")";
    },
  },
  methods: {
    mouseMove(index) {
      if (this.mouseLeftPressed) {
        let setState =
          this.eventTrigger && this.currSwitchstate == 0
            ? -1
            : this.currSwitchstate;
        this.initValue[this.indexLine]["value"][index]= setState;
      }
    },
    mouseLeftDown(index) {
      this.mouseLeftPressed = true;
      let setState =
        this.eventTrigger && this.currSwitchstate == 0
          ? -1
          : this.currSwitchstate;
      this.initValue[this.indexLine]["value"][index] = setState;
    },
    mouseLeftUp() {
      this.mouseLeftPressed = false;
    },
    _eventBackground: function (index) {
      if (this.initValue[this.indexLine].value[index] == -1) {
        return this.scaleColors[0];
      }
    },
    _range: function () {
      let start = this.currTime - 24 / this.zoomFactor / 2;
      if (start < 0) start = 0;
      if (start + 24 / this.zoomFactor > 24) start = 24 - 24 / this.zoomFactor;
      start = start * 4;
      let diff = 96 / this.zoomFactor;
      return Array(diff)
        .fill()
        .map((_, idx) => start + idx);
    },
  },
};
</script>

<style>
</style>