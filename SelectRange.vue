<script>
export default {
  name: "SelectRange",
  props: {
    options: {
      type: null,
      default: function () {
        return null;
      },
    },
    modelValue: {
      type: Number,
      default: null,
    },
    minValue: {
      type: Number,
      default: null,
    },
    maxValue: {
      type: Number,
      default: null,
    },
    step: {
      type: Number,
      default: null,
    },
    minMaxAppend: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      isMounted: false,
      min: this.minValue || (this.options ? this.options[0] : 0),
      max:
        this.maxValue ||
        (this.options ? this.options[this.options.length - 1] : 100),
      currentValue: this.modelValue || this.options[0] || 0,
      dragged: false,
      mouseOffset: 0,
    };
  },
  computed: {
    percentageValue: function () {
      if (this.max == this.min) {
        return "99%";
      }

      return (
        ((this.currentValue - this.min) / (this.max - this.min)) * 96.5 +
        1.5 +
        "%"
      );
    },
    roundedValue: function () {
      var d = Number.POSITIVE_INFINITY;
      var index = 0;

      if (!this.options) {
        const step = this.step || 1;
        return Math.round(this.currentValue / step) * step;
      }

      for (var i = 0; i < this.options.length; i++) {
        if (Math.abs(this.currentValue - this.options[i]) < d) {
          d = Math.abs(this.currentValue - this.options[i]);
          index = i;
        }
      }

      return this.options[index];
    },
    formatedValue: function () {
      return this.roundedValue.toLocaleString("ru-RU");
    },
    unitVariant: function () {
      var value = this.roundedValue;

      value = value % 100;

      if (value > 20) {
        value = value % 10;
      }

      if (value == 1) {
        return "unit1";
      } else if (value > 1 && value < 5) {
        return "unit2";
      } else {
        return "unit5";
      }
    },
  },
  watch: {
    roundedValue: function (value) {
      this.$emit("update:modelValue", value);
    },
    modelValue: function (value) {
      if (!this.dragged) {
        this.currentValue = value;
      }
    },
  },
  mounted() {
    var vm = this;
    this.isMounted = true;

    window.addEventListener("mousemove", function (event) {
      vm.mouseMove(event);
    });
    window.addEventListener("touchmove", function (event) {
      vm.mouseMove(event);
    });
    window.addEventListener("mouseup", function (event) {
      vm.mouseUp(event);
    });
    window.addEventListener("touchend", function (event) {
      vm.mouseUp(event);
    });
    window.addEventListener("touchcancel", function (event) {
      vm.mouseUp(event);
    });

    this.$emit("update:modelValue", this.roundedValue);
  },
  methods: {
    mouseDown: function (event) {
      this.dragged = true;

      if (event.touches) {
        this.mouseOffset =
          event.touches[0].pageX -
          event.target.getBoundingClientRect().left -
          20;
      } else {
        this.mouseOffset = event.layerX - 20;
      }
    },
    mouseMove: function mouseMove(event) {
      if (this.dragged) {
        var eventPageX = event.touches ? event.touches[0].pageX : event.pageX;
        var bar = this.$refs.bar;
        var barX = bar.getBoundingClientRect().left;
        var barWidth = bar.offsetWidth - 14;
        var relativeMouseX = eventPageX - barX - this.mouseOffset;

        this.currentValue =
          ((relativeMouseX - 20) / barWidth) * (this.max - this.min) + this.min;

        if (this.currentValue < this.min) {
          this.currentValue = this.min;
        }

        if (this.currentValue > this.max) {
          this.currentValue = this.max;
        }
      }
    },
    mouseUp: function mouseUp() {
      if (this.dragged) {
        //this.sendResult();
      }

      this.dragged = false;
    },
    formatValue(value) {
      const num = parseInt(value);
      const string = num.toLocaleString("ru-RU");

      return string;
    },
  },
};
</script>

<template>
  <div class="linear-selector">
    <div class="linear-selector-rail" ref="bar">
      <div
        class="linear-selector-fill"
        :style="{ width: percentageValue }"
      ></div>
      <div
        class="linear-selector-drag"
        :style="{ left: percentageValue }"
        @mousedown="mouseDown"
        @touchstart="mouseDown"
      ></div>
    </div>
    <div v-if="minValue && maxValue" class="linear-selector-min-max">
      <div class="linear-selector-min-max-value">
        {{ formatValue(minValue) }}{{ minMaxAppend }}
      </div>
      <div class="linear-selector-min-max-value">
        {{ formatValue(maxValue) }}{{ minMaxAppend }}
      </div>
    </div>
  </div>
</template>

<style lang="scss">
.linear-selector {
  padding: 19px 0 32px 0;

  .linear-selector-rail {
    position: relative;
    height: 3px;
    background: #e3e3e1;
    margin-bottom: 10px;
  }

  .linear-selector-fill {
    position: absolute;
    left: 0;
    top: 0;
    height: 3px;
    background: $a_color;
  }

  .linear-selector-drag {
    position: absolute;
    top: 1px;
    width: 40px;
    height: 40px;
    margin: -20px;
    cursor: ew-resize;
  }

  .linear-selector-drag::before,
  .linear-selector-drag::after {
    content: "";
    font-size: 0;
    color: transparent;
    display: block;
    position: absolute;
    border-radius: 50%;
  }

  .linear-selector-drag::before {
    width: 21px;
    height: 21px;
    margin: -10px;
    left: 50%;
    top: 50%;
    background: $a_color;
  }

  .linear-selector-drag::after {
    width: 13px;
    height: 13px;
    margin: -6px;
    left: 50%;
    top: 50%;
    background: $a_color;
  }

  .linear-selector-drag:hover::after {
    filter: brightness(1.1);
  }

  .linear-selector-min-max {
    display: flex;
    flex-flow: row nowrap;
    align-items: baseline;
    justify-content: space-between;
    padding-top: 3px;
  }

  .linear-selector-min-max-value {
    color: #a2a09d;
    font-size: 14px;
  }
}
</style>
