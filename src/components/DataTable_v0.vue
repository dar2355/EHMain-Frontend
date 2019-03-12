<template>
  <el-table id="table-main"
    :data="data"
    :header-cell-style="{backgroundColor: '#f5f7fa'}">
    <!-- the select button -->
    <el-table-column label="select" :width="80" key="row_selection">
      <template slot-scope="scope">
        <el-checkbox-button v-model="selected[scope.$index]">✓</el-checkbox-button>
      </template>
    </el-table-column>

    <!-- the dynamic data -->
    <el-table-column v-for="row of rowData" :key="`row_${rowData.indexOf(row)}`"
      :prop="(row.subsections) ? '' : row.name"
      :label="(row.isActive) ? row.name : null"
      :width="(row.isActive) ? null : 0"
      >

      <!-- SECTION subsections -->
      <template v-if="row.subsections !== undefined">
        <el-table-column v-for="[name, section] of Object.entries(row.subsections)" :key="`subrow_${row.name}_${name}`"
          :prop="name"
          :label="row.isActive ? name : ''"
          :width="(row.isActive) ? null : 0">
          <template slot-scope="scope">
            <dynamic-input-array v-if="section.isArray"
              :type="section.type"
              :dateKind="(section.type === 'Date') ? section.kind : null"
              :disabled="section.isDisabled"
              :active="row.isActive"
              :isEnum="section.enum && section.enum.length > 0"
              :enumValues="section.enum"
              :givenValues="scope.row[section.name]"
              :ref="`input_${scope.$index}`"
              />
            <dynamic-input v-else
              :type="section.type"
              :dateKind="(section.type === 'Date') ? section.kind : null"
              :disabled="section.isDisabled"
              :active="row.isActive"
              :isEnum="section.enum && section.enum.length > 0"
              :enumValues="section.enum"
              :givenValue="scope.row[section.name]"
              :ref="`input_${scope.$index}`"
              >
            </dynamic-input>
          </template>
        </el-table-column>
      </template>
      <!-- !SECTION -->

      <template slot-scope="scope">
        <dynamic-input-array v-if="row.isArray"
          :type="row.type"
          :dateKind="(row.type === 'Date') ? row.kind : null"
          :disabled="row.isDisabled"
          :active="row.isActive"
          :isEnum="row.enum && row.enum.length > 0"
          :enumValues="row.enum"
          :givenValues="scope.row[row.name]"
          :ref="`input_${scope.$index}`"
          />
        <dynamic-input v-else
          :type="row.type"
          :dateKind="(row.type === 'Date') ? row.kind : null"
          :disabled="row.isDisabled"
          :active="row.isActive"
          :isEnum="row.enum && row.enum.length > 0"
          :enumValues="row.enum"
          :givenValue="scope.row[row.name]"
          :ref="`input_${scope.$index}`"
          >
        </dynamic-input>
      </template>
    </el-table-column>
  </el-table>
</template>

<script lang="ts">
import Vue from 'vue';
import DynamicInputArray from '@/components/DynamicInputArray.vue';
import DynamicInput from '@/components/DynamicInput.vue';

export default Vue.extend({
  name: 'dataTable',
  components: {
    DynamicInputArray,
    DynamicInput,
  },
  props: {
    rowData: {
      default: [] as any[],
      type: Array,
    },
    data: {
      default: [] as any[],
      type: Array,
    },
    endpoint: String,
  },
  data() {
    return {
      updated: [] as any[],
      selected: [] as boolean[],
    };
  },
  watch: {
    data(newVal, oldVal) {
      console.log(newVal);
      while (this.selected.length < this.data.length) {
        this.selected.push(false);
      }
      if (this.selected.length > this.data.length) {
        this.selected.slice(0, this.data.length);
      }
    },
  },
  methods: {
    doThing(evt, index, name) {
      // this.data[index][name] = evt.target.value;
      // console.log(this.data);
    },
    getUpdates() {
      return this.updated;
    },
    async submit() {
      const elements = [];
      for (let i = 0; i < this.data.length; i++) {
        elements.push(this.$refs[`input_${i}`].map((el: any) => el.getValue()));
      }

      let selected;
      if (this.noneSelected) { selected = elements; }
      else { selected = elements.filter((_: any, i: number) => this.selected[i]); }

      const toSend = selected.map((arr: any) => {
        const outObj = {};
        for (const [i, row] of this.rowData.entries()) {
          if (row.hasOwnProperty('subsections')) {
            outObj[row.name] = {};
            for (const [subI, key] of Object.keys(row.subsections).entries()) {
              outObj[row.name][key] = arr[i + subI];
            }
          } else {
            outObj[row.name] = arr[i];
          }

        }
        return outObj;
      });

      console.log(toSend);

      const res = await fetch(this.endpoint, {
        method: 'PUT',
        headers: {
          'authorization': '' + window.sessionStorage.getItem('jwt'),
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(toSend),
      });

      console.log(await res.json());
    },
    async deleteSelected() {
      // console.log(this.updated.filter((_: any, i: number) => this.selected[i]));
      const toSend = this.splitArrays(this.updated.filter((_: any, i: number) => this.selected[i]));

      const res = await fetch(this.endpoint, {
        method: 'DELETE',
        headers: {
          'authorization': '' + window.sessionStorage.getItem('jwt'),
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(toSend),
      });

      console.log(res);
    }
  },
  computed: {
    filtered() {
      console.log(this.rowData);
      return this.rowData.filter((row: any) => row.isActive);
    },
    noneSelected() {
      return this.selected.every((status) => status === false);
    }
  },
});
</script>

<style lang="scss">
#table-main {
  max-width: 100%;
}

// SUPER hacky, but it works (at last)
#table-main > div.el-table__header-wrapper > table > thead > tr > th.el-table_1_column_13.is-hidden {
  display: none;
}

</style>
