<template>
  <div id="overlayDisable">
    <div id="overlayDisableContent">
      <div>{{ line1 }}</div>
      <div>{{ line2 }}: {{ lastState }}</div>
      <div id="reactivate" class="activation" @click="reactivate">
        <div>{{ line3 }}</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      line1: "",
      line2: "",
      line3: "",
    };
  },
  props: {
    lang: {
      type: String,
      default: "en",
    },
    lastState: String,
  },
  created: function () {
    this.line1 = prefs.languages[this.lang].inactive.line1;
    this.line2 = prefs.languages[this.lang].inactive.line2;
    this.line3 = prefs.languages[this.lang].inactive.line3;
  },
  methods: {
    reactivate() {
      this.$emit("click");
    },
  },
};
</script>

<style>
</style>