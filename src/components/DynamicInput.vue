<template>
  <!-- Enums NOTE UNTESTED -->
  <el-dropdown v-if="isEnum" placement="bottom" :hide-on-click="false">
    <span class="el-dropdown-link"> choose one... </span>
    <el-dropdown-menu slot="dropdown">
      <el-radio-group :disabled="disabled">
        <el-dropdown-item v-for="(option) in enumValues" :key="`Enum_Toggle_${option}`">
          <el-radio-button :label="option" style="display: block" v-model="value"></el-radio-button>
        </el-dropdown-item>
      </el-radio-group>
    </el-dropdown-menu>
  </el-dropdown>

  <!-- Boolean -->
  <el-checkbox-button v-else-if="active && type === 'Boolean'" v-model="value" :disabled="disabled">
    ✓
  </el-checkbox-button>

  <!-- String -->
  <el-input v-else-if="active && type === 'String'" v-model="value" :disabled="disabled"></el-input>

  <!-- Date -->
  <el-date-picker v-else-if="active && type === 'Date'" v-model="value" :type="dateKind" :placeholder="dateKind" :disabled="disabled" style="width: 100%;"></el-date-picker>

  <!-- ObjectID -->
  <el-input v-else-if="active && type === 'ObjectID'" v-model="value" :disabled="disabled"></el-input>
</template>

<script lang="ts">
import Vue from 'vue';
export default Vue.extend({
  name: 'DynamicInput',
  props: {
    type: {
      type: String,
      default: 'String',
      validator(v: any) { return ['Boolean', 'String', 'Date', 'Time', 'DateTime', 'ObjectID'].includes(v); },
    },
    dateKind: {
      type: String,
      default: 'date',
      validator(v: any) {
        return ['year', 'month', 'date', 'dates', 'datetime', 'week', 'datetimerange', 'daterange'].includes(v);
      },
    },
    validation: {
      type: String,
    },
    isEnum: {
      type: Boolean,
      default: false,
    },
    enumValues: {
      type: Array,
      default: () => [],
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    active: {
      type: Boolean,
      default: true,
    },
    givenValue: {
      type: [String, Boolean, Number],
      default: '',
    },
  },
  data() {
    return {
      value: undefined as any,
    };
  },
  created() {
    if (this.givenValue === '') { this.value = null }
    else { this.value = this.givenValue }
  },
  methods: {
    getValue() {
      if (this.value === null || this.value.length === 0) { return ''; }
      if (['Date', 'Time', 'DateTime'].includes(this.type) && this.value.getTime) { return this.value.getTime(); }

      return this.value;
    },
    validate() {
      return (this.validation) ? new RegExp(this.validation).test(this.value) : true;
    },
  },
});
</script>
