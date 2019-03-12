<template>
  <div id="split-view">
    <div class="left">
      <data-table
        :rowData="table.rows"
        :data="table.data"
        :references="references"
        endpoint="http://127.0.0.1:3000/api/members/"
        ref="table"
        />
    </div>
    <div class="right">
      <h3>control panel</h3>
      <el-form>
        <el-form-item>
          <el-button-group>
            <el-button>Select All</el-button>
            <el-button>Deselect All</el-button>
          </el-button-group>
        </el-form-item>
        <el-form-item>
          <el-button-group>
            <el-button @click="addEntry">Add Entry</el-button>
            <el-button @click="deleteSelected" type="danger">Delete Selected</el-button>
          </el-button-group>
        </el-form-item>
        <el-form-item size="mini">
          <el-checkbox v-for="row of table.rows" v-model="row.isActive" :key="`row_toggle_${row.name}`" style="display: block; text-align: left; margin-left: 30px;">
            {{ row.name }}
          </el-checkbox>
        </el-form-item>
        <el-form-item>
          <el-button @click="submit">Save</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import DataTable from '@/components/DataTable.vue';

interface IRowData {
  name: string;
  active: boolean;
  editable: boolean;
  array: boolean;
  required: boolean;
  type: string;
  reference: string;
  enum?: string[];
}

interface IStructureData {
  getters: any[];
  instance: string;
  options: any;
  path: string;
  setters: any[];
  validators: any[];
  caster?: IStructureData;
}

export default Vue.extend({
  name: 'users',
  components: {
    DataTable,
  },
  data() {
    return {
      table: {
        rows: [] as any[],
        data: [] as any[],
      },
      references: {
        members: [] as any[],
      },
    };
  },
  async mounted() {
    let res = await fetch('http://127.0.0.1:3000/api/members/full', {
      method: 'GET',
      headers: {
        authorization: '' + window.sessionStorage.getItem('jwt'),
      },
    });
    const memberDataFull = await res.json();

    res = await fetch('http://127.0.0.1:3000/api/members/simple', {
      method: 'GET',
      headers: {
        authorization: '' + window.sessionStorage.getItem('jwt'),
      },
    });
    const memberDataSimple = await res.json();

    this.table.data = memberDataFull.members;
    this.table.rows = memberDataFull.structure;
    this.references.members = memberDataSimple.members;
    console.log(memberDataFull);
  },
  methods: {
    addEntry() {
      //
    },
    submit() {
      this.$refs.table.submit('http://127.0.0.1:3000/api/members');
    },
    deleteSelected() {
      this.$refs.table.deleteSelected('http://127.0.0.1:3000/api/members');
    },
  },
});
</script>

<style lang="scss" scoped>
  #split-view {
    display: flex;
    flex-direction: row;
    width: 100%;

    .left {
      flex: 1;
      max-width: calc(100% - 300px);
    }
    .right {
      width: 300px;
      flex-shrink: 0;
    }
  }
</style>
