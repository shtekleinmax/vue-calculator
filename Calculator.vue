<script>
import axios from "@/init/axios.js";

export default {
  name: "Caclulator",
  props: {
    action: {
      type: String,
      default: "",
    },
    cities: {
      type: Object,
      default: function () {
        return {};
      },
    },
  },
  data() {
    return {
      fields: {
        amount: parseInt(this.$v("CALC_MIN_AMOUNT")),
        term: parseInt(this.$v("CALC_MIN_TERM")),
        payment: "",
        name: "",
        phone: "",
        city: {
          title: "",
          value: "",
        },
        brand: "",
        model: "",
        position: "",
        year: "",
        sms_agree: false,
      },
      minAmount: parseInt(this.$v("CALC_MIN_AMOUNT")),
      maxAmount: parseInt(this.$v("CALC_MAX_AMOUNT")),
      stepAmount: parseInt(this.$v("CALC_STEP_AMOUNT")),
      minTerm: parseInt(this.$v("CALC_MIN_TERM")),
      maxTerm: parseInt(this.$v("CALC_MAX_TERM")),
      stepTerm: parseInt(this.$v("CALC_STEP_TERM")),
      confirm: false,
      success: false,
      loading: false,
    };
  },
  computed: {
    interestRate: function () {
      return parseFloat(this.$v("CALC_INTEREST_RATE")) / 100;
    },
    totalPayment: function () {
      return this.monthlyPayment * this.fields.term;
    },
    monthlyPayment: function () {
      return this.getPayment();
    },
    citiesList() {
      let cities = this.cities.map(function (city) {
        return { id: city.id, title: city.title };
      });

      return JSON.stringify(cities);
    },
  },
  methods: {
    getPayment() {
      this.fields.payment = Math.round(
        this.calculatePMT(
          this.interestRate,
          this.fields.term,
          this.fields.amount
        )
      );

      return this.fields.payment;
    },
    calculatePMT(
      ratePerPeriod,
      numberOfPayments,
      presentValue,
      futureValue,
      type
    ) {
      futureValue = typeof futureValue !== "undefined" ? futureValue : 0;
      type = typeof type !== "undefined" ? type : 0;

      if (ratePerPeriod > 0) {
        let q = Math.pow(1 + ratePerPeriod, numberOfPayments);
        return (
          (ratePerPeriod * (futureValue + q * presentValue)) /
          ((-1 + q) * (1 + ratePerPeriod * type))
        );
      } else if (numberOfPayments > 0) {
        return (futureValue + presentValue) / numberOfPayments;
      }

      return 0;
    },
    formatValue(value) {
      const num = parseInt(value);
      const string = num.toLocaleString("ru-RU");

      return string;
    },
    validate() {
      let result = true;
      this.success = true;
      let fields = Object.entries(this.fields);

      if (this.fields && fields.length > 0) {
        for (const field in this.fields) {
          if (field == "payment") {
            continue;
          }

          this.$refs[field].firstInput = false;

          if (!this.$refs[field].validate()) {
            result = false;
          }
        }
      }

      return result;
    },
    submit() {
      if (!this.validate()) {
        return;
      }

      const action = this.action;
      const sendData = Object.assign({}, window.config, {
        form: this.fields,
      });

      this.loading = true;

      axios.post(action, sendData).then((response) => {
        this.loading = false;

        if (response) {
          if (response.result === "success") {
            if (this.clear) {
              this.clearForm();
            }
          }

          if (response.error) {
            this.$root.showModal("modal_message", false, response.error);
          }

          if (response.msg) {
            this.$root.showModal(
              "modal_message",
              false,
              response.msg,
              response.location
            );
          } else if (response.location) {
            window.location = response.location;
          }
        }
      });
    },
  },
};
</script>

<template>
  <transition name="fast-fade">
    <div v-if="loading" class="overlay progress"></div>
  </transition>

  <div class="calculator">
    <div class="controllers">
      <div class="caption-field-row">
        <div class="caption">{{ $v("STR_CREDIT_AMOUNT") }}</div>
        <number-field
          :ref="'amount'"
          v-model="fields.amount"
          :append="$v('STR_CURRENCY')"
          :min="minAmount"
          :max="maxAmount"
          :width="130"
        />
      </div>

      <linear-selector
        v-model="fields.amount"
        :minValue="minAmount"
        :maxValue="maxAmount"
        :step="stepAmount"
        :minMaxAppend="' ' + $v('STR_CURRENCY')"
      />
      <div class="caption-field-row">
        <div class="caption">{{ $v("STR_LOAN_TERMS") }}</div>
        <number-field
          :ref="'term'"
          v-model="fields.term"
          :append="$v('STR_MONTH_SHORT')"
          :min="minTerm"
          :max="maxTerm"
          :width="90"
        />
      </div>

      <linear-selector
        v-model="fields.term"
        :minValue="minTerm"
        :maxValue="maxTerm"
        :value="fields.term"
        :step="stepTerm"
        :minMaxAppend="' ' + $v('STR_MONTH_SHORT')"
      />

      <div class="total-row">
        <div class="caption">{{ $v("STR_MONTHLY_PAYMENT") }}:</div>
        <div class="value">
          {{ formatValue(monthlyPayment) }} {{ $v("STR_CURRENCY") }}
        </div>
      </div>

      <div class="note">
        <div class="caption">{{ $v("STR_CALCULATION_IS_PRELIMINARY") }}</div>
        <div class="value">
          {{ $v("STR_CALCULATION_NOTE") }}
        </div>
      </div>
    </div>

    <div class="form">
      <div class="form-field text">
        <text-field
          :ref="'name'"
          :label="$v('STR_FIO')"
          :type="'text'"
          :required="true"
          v-model="fields.name"
        />
      </div>

      <div class="form-field tel">
        <text-field
          :ref="'phone'"
          :label="$v('STR_PHONE_NUMBER')"
          :type="'tel'"
          v-model="fields.phone"
          mask="(99) 999-99-99"
          prefix="+998 "
          :inputmode="'tel'"
          :required="true"
        />
      </div>

      <div class="form-field select">
        <select-field
          :ref="'city'"
          :label="$v('STR_CITY')"
          :options="citiesList"
          :required="true"
          v-model="fields.city.value"
          :title="fields.city.title"
          @update:title="fields.city.title = $event"
        />
      </div>

      <div class="form-field text">
        <text-field
          :ref="'brand'"
          :label="$v('STR_CAR_BRAND')"
          :type="'text'"
          v-model="fields.brand"
          :required="true"
        />
      </div>

      <div class="form-field text">
        <text-field
          :ref="'model'"
          :label="$v('STR_CAR_MODEL')"
          :type="'text'"
          v-model="fields.model"
          :required="true"
        />
      </div>

      <div class="form-field text">
        <text-field
          :ref="'position'"
          :label="$v('STR_POSITION')"
          :type="'text'"
          v-model="fields.position"
          :required="true"
        />
      </div>

      <div class="form-field text">
        <text-field
          :ref="'year'"
          :label="$v('STR_PRODUCTION_YEAR')"
          :type="'text'"
          v-model="fields.year"
          :inputmode="'numeric'"
          :required="true"
        />
      </div>

      <div class="form-field">
        <checkbox-field
          :ref="'sms_agree'"
          :label="$v('STR_SMS_AGREE_TEXT')"
          :type="'checkbox'"
          v-model="fields.sms_agree"
          :required="true"
          :errortext="{
            required: $v('MSG_REQUIRED_FIELD'),
          }"
        />
      </div>

      <div class="form-send">
        <button type="button" class="button primary" @click="submit">
          {{ $v("STR_SEND") }}
        </button>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
.calculator {
  .caption-field-row {
    display: flex;
    flex-flow: row nowrap;
    align-items: center;
    justify-content: flex-start;
  }

  .caption {
    flex-grow: 1;
    flex-shrink: 1;
    font-size: 16px;
    font-weight: 600;
  }

  .note {
    margin-top: 20px;
    text-align: center;

    .caption {
      margin-bottom: 18px;
      font-size: 18px;
      font-weight: 600;
      color: $a_color;
    }

    .value {
      font-size: 14px;
      color: #a2a09d;
    }
  }
}

.total-row {
  margin-top: 7px;
  padding: 14px 25px;
  background: #f4f4f4;
  border-radius: 5px;
  font-size: $font_size;
  font-weight: 400;
  color: $font_color;

  .caption {
    margin-bottom: 4px;
    font-size: 16px;
    font-weight: 400;
    text-align: center;
  }

  .value {
    font-size: 30px;
    font-weight: 700;
    line-height: 1.2;
    text-align: center;
    color: $font_color;
  }
}
</style>
