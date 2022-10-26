<script>
export default {
  props: {
    label: {
      type: String,
      default: "",
    },
    options: {
      type: String,
      default: "",
    },
    defaultoption: {
      type: Number,
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
    title: {},
    modelValue: {},
  },
  data() {
    return {
      error: false,
      success: false,
      showOptions: false,
      valueData: this.modelValue,
      valueTitle: this.title,
      errorMsg: "",
    };
  },
  watch: {
    valueData(value) {
      this.$emit("update:modelValue", value);
    },
    valueTitle(title) {
      this.$emit("update:title", title);
    },
    options(value) {
      if (value) {
        this.selectValue(JSON.parse(value));
      }
    },
  },
  methods: {
    selectValue(option) {
      this.valueData = option.id;
      this.valueTitle = option.title;
      this.showOptions = false;
      this.validate();
    },
    changeOptions() {
      if (this.options.length > 0) {
        this.showOptions = !this.showOptions;
        this.success = false;
        this.error = false;
      }
    },
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
    isActive() {
      return this.valueData;
    },
  },
  mounted() {
    if (this.defaultoption && this.options) {
      this.valueTitle = this.options[this.defaultoption].title;
      this.valueData = this.defaultoption;
    }
  },
};
</script>

<template>
  <div v-if="showOptions" class="select-overlay" @click="changeOptions()"></div>

  <label
    class="input-wrap"
    :class="{
      error: error,
      success: success,
      active: isActive,
    }"
  >
    <span class="label unselectable" v-if="label">{{ label }}</span>

    <span class="textbox select-wrap" @click="changeOptions()">
      <span class="selectbox-value">
        <span class="value-title">{{ valueTitle }}</span>
      </span>

      <svg-icon :icon="showOptions ? 'select_top' : 'arrow_select'" />

      <input type="hidden" ref="input" v-model="valueData" />
    </span>

    <span class="error-msg" v-if="error"> {{ errorMsg }} </span>
  </label>

  <div class="selectbox-wrap" v-if="showOptions">
    <div class="scorll-wrap">
      <ul class="selectbox-list">
        <li
          v-for="option in JSON.parse(this.options)"
          :key="option.id"
          :class="{ selected: option.id === valueData }"
          @click="selectValue(option)"
        >
          <span class="option">{{ option.title }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<style lang="scss">
/*
.multiselect {
    border: 1px solid $form_textbox_border_color;
    border-radius: 3px;
}
*/
.form-field {
  &.select {
    z-index: 2;

    .select-wrap {
      padding: 26px 0 0 0;
      background: transparent;
    }

    .input-wrap {
      background: #fff;
      text-align: left;
    }
  }

  &.disabled {
    opacity: 0.5;
    position: relative;
    overflow: hidden;

    &::before {
      content: "";
      width: 100%;
      height: 100%;
      display: block;
      background: transparent;
      position: absolute;
      top: 0;
      left: 0;
      z-index: 2;
    }
  }
}

.select-overlay {
  position: fixed;
  width: 100vw;
  height: 100vh;
  background: transparent;
  left: 0;
  top: 0;
  z-index: 4;
}

.select-wrap {
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-between;
  background: transparent;
  position: relative;
  cursor: pointer;
  z-index: 2;

  .icon {
    width: 14px;
    height: 14px;
    background: #fff;
    margin-right: 4px;
    padding: 0;
  }
}

.selectbox-wrap {
  max-width: 360px;
  width: 100%;
  margin-top: 10px;
  padding: 21px 21px;
  background: #fff;
  box-shadow: 3px 7px 20px rgba(242, 65, 57, 0.15);
  border-radius: 5px;
  position: absolute;
  z-index: 5;

  .option {
    font-size: $font_size;
    font-weight: $font_weight;
    color: $font_color;
    cursor: pointer;

    &:hover {
      color: $a_color;
    }
  }
}

.scorll-wrap {
  max-height: 98px;
  overflow-x: hidden;
  overflow-y: scroll;
  scrollbar-color: #32363c transparent;
  scrollbar-width: thin;

  &::-webkit-scrollbar-track {
    background-color: transparent;
  }

  &::-webkit-scrollbar-thumb {
    border-radius: 5px;
    background: #32363c;
  }

  &::-webkit-resizer {
    width: 7px;
    height: 0px;
  }

  &::-webkit-scrollbar {
    width: 7px;
  }
}

.selectbox-list {
  list-style: none;

  li {
    padding: 5px 15px;
    display: flex;
    flex-flow: row nowrap;
    align-items: center;
  }
}

.value-title {
  margin-top: 5px;
  display: none;
}

.active {
  .value-title {
    display: block;
  }
}

.selectbox-value {
  overflow: hidden;
}
</style>
