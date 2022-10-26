<script>
export default {
  props: {
    label: {
      type: String,
      default: "",
    },
    modelValue: {
      type: String,
      default: "",
    },
    maxLength: {
      type: Number,
      default: 2000,
    },
    minLength: {
      type: Number,
      default: 1,
    },
    placeholder: {
      type: String,
      default: "",
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
  },
  data() {
    return {
      error: false,
      success: false,
      isFocused: false,
      firstInput: true,
      valueData: this.modelValue,
      errors: "",
      errorMsg: "",
    };
  },
  watch: {
    modelValue(value) {
      this.valueData = value;

      if (this.validate()) {
        this.firstInput = false;
      }
    },
    valueData(newValue) {
      this.error = false;
      this.success = false;

      let value = newValue;

      if (value && this.maxLength) {
        value = value.substr(0, this.maxLength);
      }

      if (value !== newValue) {
        this.valueData = value;
      } else {
        this.$emit("update:modelValue", newValue);
      }
    },
  },
  computed: {
    isActive() {
      return this.isFocused || this.modelValue;
    },
  },
  methods: {
    focus() {
      this.isFocused = true;
    },
    blur() {
      this.isFocused = false;
      this.firstInput = false;
      this.validate();
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

      if (this.modelValue.length < this.minLength) {
        this.error = true;
        this.errorMsg = this.errortext["tooShort"];
        return false;
      }

      this.error = false;
      return (this.success = true);
    },
  },
};
</script>

<template>
  <label
    class="input-wrap"
    :class="{
      active: isActive,
      error: error && !firstInput,
      success: success,
    }"
  >
    <span class="label" v-if="label">{{ label }}</span>
    <textarea
      class="textarea"
      v-model="valueData"
      @focus="focus"
      @blur="blur"
    ></textarea>
    <span class="error-msg" v-if="error && !firstInput">{{ errorMsg }}</span>
  </label>
</template>

<style lang="scss">
.form-field {
  &.text-area {
    .active {
      .label {
        width: calc(100% - 8px);
        background: #fff;
      }
    }
  }
}

.textarea {
  width: 100%;
  min-height: 100px;
  margin: 0;
  padding: $form_textbox_padding;
  font-size: $font_size;
  font-weight: $font_weight;
  color: $font_color;
  vertical-align: top;
  background-color: #fff;
  outline: none;
  box-sizing: border-box;
  -webkit-appearance: none;
  -moz-appearance: none;
  -ms-appearance: none;
  -o-appearance: none;
  appearance: none;
  resize: none;
  overflow-x: hidden;
  overflow-y: scroll;
  scrollbar-color: $font_color transparent;
  scrollbar-width: thin;
  border: 0;
  border-bottom: 1px solid $form_textbox_border_color;
  border-radius: 0;

  &:focus {
    box-shadow: none;
  }

  &::-webkit-scrollbar-track {
    background-color: transparent;
  }

  &::-webkit-scrollbar-thumb {
    border-radius: 5px;
    background: $font_color;
  }

  &::-webkit-resizer {
    width: 7px;
    height: 0px;
  }

  &::-webkit-scrollbar {
    width: 7px;
  }
}
</style>
