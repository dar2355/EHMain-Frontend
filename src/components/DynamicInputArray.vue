<template>
  <el-dropdown placement="bottom" :hide-on-click="false">
    <span class="el-dropdown-link"> {{active ? (isEnum ? 'select...' : 'input...') : ''}} </span>
    <el-dropdown-menu slot="dropdown">
      <!-- Enums -->
      <template v-if="isEnum">
        <el-dropdown-item v-for="(option, index) in enumValues" :key="`Enum_Toggle_${option}`">
          <el-checkbox :label="option" style="display: block" v-model="values[index]"></el-checkbox>
        </el-dropdown-item>
      </template>

      <!-- Booleans -->
      <template v-else-if="type === 'Boolean'">
        <el-dropdown-item v-for="(label, index) in givenLabels" :key="`Bool_Toggle_${label}`">
          <el-checkbox :label="label" style="display: block" v-model="values[index]"></el-checkbox>
        </el-dropdown-item>
      </template>

      <!-- Strings -->
      <template v-else-if="type === 'String'">
        <el-dropdown-item v-for="(value, index) in values" :key="`String_Index_${index}`">
          <el-input class="input-string" v-model="values[index]">
            <el-button slot="prepend" @click="() => {remove(index)}">-</el-button>
          </el-input>
        </el-dropdown-item>
        <el-dropdown-item>
          <el-button size="mini" @click="append">add</el-button>
        </el-dropdown-item>
      </template>

      <!-- TODO Date -->


      <!-- ObjectIDs -->
      <template v-else-if="type === 'ObjectID' && referenced">
        <el-dropdown-item v-for="(option, index) in referencedRelevant" :key="`ObjectID_selector_${index}`">
          <el-checkbox :label="Object.values(option).join(', ')" style="display: block" v-model="values[index]"></el-checkbox>
        </el-dropdown-item>
      </template>
    </el-dropdown-menu>
  </el-dropdown>
</template>

<script lang="ts">
import Vue from 'vue';
export default Vue.extend({
  name: 'DynamicInputArray',
  props: {
    type: {
      type: String,
      default: 'String',
      validator(v: any) { return ['Boolean', 'String', 'Date', 'Time', 'DateTime', 'ObjectID'].includes(v); },
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
    givenLabels: {
      type: Array,
      default: () => [],
    },
    givenValues: {
      type: Array,
      default: () => [],
    },
    validation: {
      type: String,
      default: null,
    },
    referenced: {
      type: Array,
      default: null,
    },
  },
  data() {
    return {
      values: [] as any[],
    };
  },
  created() {
    if (this.isEnum) {
      this.values = this.enumValues.map(option => this.givenValues.includes(option));
    } else if (this.type === 'ObjectID' && this.referenced) {
      this.values = this.referenced.map(option => this.givenValues.includes(option._id));
    } else {
      this.values = (this as any).givenValues || [];
    }
  },
  methods: {
    append() {
      this.values.push('');
    },
    remove(index: number) {
      this.values.splice(index, 1);
    },
    validate(): boolean {
      if (this.validation) {
        return this.values.every((val) => new RegExp(this.validation).test(val));
      } else {
        return true;
      }
    },
    getValue() {
      if (this.isEnum) {
        if (!this.values || this.values.length === 0) return [];
        return this.enumValues.filter((_, i) => this.values[i]);
      }

      if (this.referenced) {
        if (!this.values || this.values.length === 0) return [];
        return this.referenced.filter((_, i) => this.values[i]).map(obj => obj._id);
      }

      return this.values;
    },
  },
  computed: {
    referencedRelevant() {
      return this.referenced.map((entry) => {
        const clone = JSON.parse(JSON.stringify(entry));
        delete clone._id;
        return clone;
      });
    },
  },
});
</script>