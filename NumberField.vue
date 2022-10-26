<script>
export default {
  name: "NumberField",
  props: {
    modelValue: {
      type: Number,
      default: null,
    },
    min: {
      type: Number,
      default: Number.NEGATIVE_INFINITY,
    },
    max: {
      type: Number,
      default: Number.POSITIVE_INFINITY,
    },
    prepend: {
      type: null,
      default: null,
    },
    append: {
      type: String,
      default: null,
    },
    width: {
      type: Number,
      default: 150,
    },
  },
  data() {
    return {
      textValue: this.formatValue(this.clampValue(this.modelValue)),
      error: false,
      success: false,
      errors: "",
      errorMsg: "",
    };
  },

  computed: {
    style: function () {
      return {
        width: this.width + "px",
      };
    },
  },
  watch: {
    modelValue: function (value) {
      this.textValue = this.formatValue(this.clampValue(value));
    },
  },
  methods: {
    clampValue(value) {
      if (value > this.max) {
        return this.max;
      }

      if (value < this.min) {
        return this.min;
      }

      return value;
    },
    formatValue(value) {
      const num = parseInt(value);
      const string = num.toLocaleString("ru-RU");

      return string;
    },
    onBlur: function () {
      const value = this.clampValue(
        parseInt(this.textValue.replace(/[^\d.-]/g, "")) || 0
      );
      this.textValue = this.formatValue(value);
      this.$emit("update:modelValue", value);
    },
    validate() {
      this.success = false;
      this.error = false;
      this.errorMsg = "";

      if (!this.modelValue) {
        if (this.required) {
          this.error = true;
          this.errorMsg = this.errortext["required"];
          return false;
        }

        return true;
      }

      return (this.success = true);
    },
  },
};
</script>

<template>
  <label class="field number-field" :style="style">
    <div v-if="prepend" class="text prepend">{{ prepend }}</div>
    <div class="input-wrap">
      <input ref="input" class="input" v-model="textValue" @blur="onBlur" />
    </div>
    <div v-if="append" class="text append">{{ append }}</div>
  </label>
</template>

<style lang="scss">
.number-field {
  height: 40px;
  padding: 0 10px 0 10px;
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  justify-content: flex-start;
  background: #fff;
  border: 1px solid #e3e3e1;
  border-radius: 3px;

  .text {
    flex-grow: 0;
    flex-shrink: 0;
    color: #a2a09d;
    font-size: 14px;
  }

  .prepend {
    margin-right: 5px;
  }

  .append {
    margin-left: 5px;
  }

  .input {
    width: 100%;
    padding: 0;
    margin: 0;
    font-size: 14px;
    color: $font_color;
    border: none;
  }
}
</style>
