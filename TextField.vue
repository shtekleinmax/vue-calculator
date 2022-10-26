<script>
import Inputmask from "inputmask";

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
      default: 50,
    },
    minLength: {
      type: Number,
      default: 1,
    },
    phoneLength: {
      type: Number,
      default: 9,
    },
    type: {
      type: String,
      default: "text",
    },
    mask: {
      type: String,
      default: "",
    },
    placeholder: {
      type: String,
      default: "",
    },
    inputmode: {
      type: String,
      default: "text",
    },
    prefix: {
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
      showPassword: false,
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

      if (this.type === "email") {
        let valid =
          /^[A-Za-z0-9!#$%&'*+-/=?^_`{|}~]+@[A-Za-z0-9-]+(\.[A-Za-z0-9-]+)+$/.test(
            this.modelValue
          );

        if (!valid) {
          this.error = true;
          this.errorMsg = this.errortext["invalidEmail"];
          return false;
        }
      } else if (this.type === "tel") {
        let valid = this.modelValue.replace(/[^0-9]/g, "");

        if (!valid || valid.length < this.phoneLength) {
          this.error = true;
          this.errorMsg = this.errortext["invalidPhone"];
          return false;
        }
      } else if (this.type === "password" || this.showPassword === true) {
        let valid = this.modelValue.length > 5;

        if (!valid) {
          this.error = true;
          this.errorMsg = this.errortext["tooShortPass"];
          return false;
        }
      }

      this.errorMsg = "";
      this.error = false;
      return (this.success = true);
    },
  },
  mounted() {
    if (this.mask) {
      Inputmask({
        mask: this.mask,
        showMaskOnHover: false,
        showMaskOnFocus: true,
      }).mask(this.$refs.input);
    }
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
    <span class="label unselectable" v-if="label">{{ label }}</span>
    <span class="prefix" v-if="prefix">{{ prefix }}</span>
    <input
      :type="showPassword ? 'text' : type"
      ref="input"
      class="textbox"
      v-model="valueData"
      @focus="focus"
      @blur="blur"
      :placeholder="isActive ? placeholder : ''"
      :inputmode="inputmode"
    />

    <div
      class="show-pass"
      v-if="type == 'password'"
      @click="showPassword = !showPassword"
    >
      <svg-icon :icon="showPassword ? 'show_pass' : 'pass'" />
    </div>

    <span class="error-msg" v-if="error && !firstInput"> {{ errorMsg }} </span>
  </label>

  <slot name="note"></slot>
</template>

<style lang="scss">
.textbox {
  width: 100%;
  margin: 0;
  padding: $form_textbox_padding;
  font-size: $font_size;
  font-weight: $font_weight;
  color: $font_color;
  vertical-align: top;
  background: #fff;
  outline: none;
  box-sizing: border-box;
  -webkit-appearance: none;
  -moz-appearance: none;
  -ms-appearance: none;
  -o-appearance: none;
  appearance: none;
  border: 0;
  border-bottom: 1px solid $form_textbox_border_color;
  border-radius: 0;

  &:focus {
    box-shadow: none;
  }
}

.prefix {
  display: none;
  position: absolute;
  left: 0;
  bottom: 6px;
  font-size: 16px;
  font-weight: 400;
  color: #32363c;
}

.active {
  & > .prefix {
    display: block;
  }
}

input[type="tel"] {
  padding-left: 42px;
}
</style>
