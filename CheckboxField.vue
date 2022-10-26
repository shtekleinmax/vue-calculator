<script>
export default {
  props: {
    label: {
      type: String,
      default: "",
    },
    value: {
      type: Boolean,
      default: false,
    },
    required: {
      type: Boolean,
      default: false,
    },
    errortext: {
      type: Object,
      default: function () {
        return {};
      },
    },
    modelValue: {},
  },
  data() {
    return {
      error: false,
      success: false,
      valueData: this.modelValue,
      errorMsg: "",
    };
  },
  watch: {
    valueData(value) {
      this.error = false;
      this.success = false;
      this.$emit("update:modelValue", value);
    },
  },
  methods: {
    validate() {
      if (!this.valueData && this.required) {
        this.error = true;
        this.errorMsg = this.errortext["required"];
        return false;
      } else if (!this.valueData) {
        return true;
      }

      this.error = false;
      return (this.success = true);
    },
  },
  computed: {
    isChecked() {
      return this.valueData;
    },
  },
};
</script>

<template>
  <label
    class="checkbox"
    :class="{
      error: error,
      success: success,
      checked: isChecked,
    }"
  >
    <span class="checkbox-title unselectable" v-if="label">{{ label }}</span>

    <input type="checkbox" v-model="valueData" />

    <span class="error-msg" v-if="error"> {{ errorMsg }} </span>
  </label>
</template>

<style lang="scss">
.checkbox {
  margin: 30px 0 5px 0;
  display: flex;
  flex-flow: column nowrap;
  position: relative;

  &-title {
    padding-left: 30px;
    font-size: $font_size;
    line-height: 20px;
    color: $font_color;
    cursor: pointer;
  }

  &::before {
    content: "";
    width: 20px;
    height: 20px;
    display: block;
    background: #fff;
    position: absolute;
    left: 0;
    top: 0;
    border: 1px solid #e3e3e1;
    border-radius: 3px;
    overflow: hidden;
    z-index: 1;
    cursor: pointer;
  }

  &::after {
    content: "";
    width: 10px;
    height: 10px;
    display: block;
    background: #de2127;
    position: absolute;
    opacity: 0;
    border-radius: 2px;
    left: 5px;
    top: 5px;
    z-index: 2;
    cursor: pointer;
  }

  &.checked {
    &::after {
      opacity: 1;
    }
  }

  &.error {
    &::before {
      border-color: $error;
    }
  }

  /*
  &.success {
    &::before {
      border-color: $success;
    }
  }
*/

  input {
    width: 0;
    height: 0;
    opacity: 0;
    position: absolute;
  }
}
</style>
