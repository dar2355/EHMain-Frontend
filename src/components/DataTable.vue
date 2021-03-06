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
        <el-table-column v-for="section of row.subsections" :key="`subrow_${row.name}_${section.name}`"
          :prop="section.name"
          :label="row.isActive ? section.name : ''"
          :width="(row.isActive) ? null : 0">
          <template slot-scope="scope">
            <dynamic-input-array v-if="section.isArray"
              :type="section.type"
              :dateKind="(section.type === 'Date') ? section.kind : null"
              :disabled="section.isDisabled"
              :active="section.isActive"
              :isEnum="section.enum && section.enum.length > 0"
              :enumValues="section.enum"
              :givenValues="scope.row[row.name][section.name]"
              :ref="`input_${scope.$index}_${row.name}_${section.name}`"
              />
            <dynamic-input v-else
              :type="section.type"
              :dateKind="(section.type === 'Date') ? section.kind : null"
              :disabled="section.isDisabled"
              :active="section.isActive"
              :isEnum="section.enum && section.enum.length > 0"
              :enumValues="section.enum"
              :givenValue="scope.row[row.name][section.name]"
              :ref="`input_${scope.$index}_${row.name}_${section.name}`"
              />
          </template>
        </el-table-column>
      </template>
      <!-- !SECTION -->

      <template slot-scope="scope">
        <dynamic-input-array v-if="row.isArray"
          :referenced="references[row.references]"
          :type="row.type"
          :dateKind="(row.type === 'Date') ? row.kind : null"
          :disabled="row.isDisabled"
          :active="row.isActive"
          :isEnum="row.enum && row.enum.length > 0"
          :enumValues="row.enum"
          :givenValues="scope.row[row.name]"
          :ref="`input_${scope.$index}_${row.name}`"
          />
        <dynamic-input v-else
          :type="row.type"
          :dateKind="(row.type === 'Date') ? row.kind : null"
          :disabled="row.isDisabled"
          :active="row.isActive"
          :isEnum="row.enum && row.enum.length > 0"
          :enumValues="row.enum"
          :givenValue="scope.row[row.name]"
          :ref="`input_${scope.$index}_${row.name}`"
          />
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
    references: {
      default: {},
      type: Object,
    },
  },
  data() {
    return {
      updated: [] as any[],
      selected: [] as boolean[],
    };
  },
  watch: {
    data(newVal, oldVal) {
      while (this.selected.length < this.data.length) {
        this.selected.push(false);
      }
      if (this.selected.length > this.data.length) {
        this.selected.slice(0, this.data.length);
      }
    },
  },
  methods: {
    getData() {
      // entries should consist of the data following
      // the given schema's structure (rowData)
      const outArray = [];
      console.log(this.$refs);

      // recursive part of this function ---
      // independent of which data entry
      // the loop below is on
      const traverseData = (index: number, rowData: any, branch: string = '') => {
        const out: any = {};
        for (const value of rowData) {
          if (value.hasOwnProperty('subsections')) {
            out[value.name] = traverseData(index, value.subsections, `${branch}_${value.name}`);
          } else {
            console.log(index, branch, value.name);
            // console.log(this.$refs[`input_${index}${branch}_${value.name}`]);
            out[value.name] = this.$refs[`input_${index}${branch}_${value.name}`][0].getValue();
          }
        }
        return out;
      };

      // loop through each column of data
      for (let i = 0; i < this.data.length; i++) {
        outArray.push(traverseData(i, this.rowData));
      }

      return outArray;
    },
    async addEntry(endpoint: string) {
      const res = await fetch(endpoint, {
        method: 'POST',
        headers: {
          'authorization': '' + window.sessionStorage.getItem('jwt'),
        },
      });

      return (await res.json()).newUser;
    },
    async submit(endpoint: string) {
      const toSend = this.getData();
      console.log(toSend);

      const res = await fetch(endpoint, {
        method: 'PUT',
        headers: {
          'authorization': '' + window.sessionStorage.getItem('jwt'),
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(toSend),
      });

      console.log(await res.json());
    },
    async deleteSelected(endpoint: string) {
      if (this.noneSelected) { return; }

      const toSend = this.getData();

      const res = await fetch(endpoint, {
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
