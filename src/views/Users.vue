<template>
  <div id="split-view">
    <div class="left">
      <data-table :rows="table.rows" :data="table.data"/>
    </div>
    <div class="right">
      <h3>control panel</h3>
      <el-checkbox v-for="row of table.rows" v-model="row.active" :key="`row_toggle_${row.name}`" style="display: block; text-align: left; margin-left: 30px;">
        {{ row.name }}
      </el-checkbox>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import DataTable from '@/components/DataTable.vue';

export default Vue.extend({
  name: 'users',
  components: {
    DataTable,
  },
  data() {
    return {
      table: {
        rows: [
          {active: false, name: 'id', dataSource: '_id'},
          {active: true, name: 'username', required: true, editable: true},
          {active: false, name: 'password'},
          {active: true, name: 'email', subsections: [
            {name: 'address', editable: true, type: 'email'},
            {name: 'verified', editable: true, type: 'boolean'},
          ]},
          {active: true, name: 'member', type: 'link', ref: '/api/members/:memberID'},
        ],
        data: [],
      },
    };
  },
  async mounted() {
    const res = await fetch('http://127.0.0.1:3000/api/users', {
      method: 'GET',
      headers: {
        authorization: '' + window.sessionStorage.getItem('jwt'),
      },
      credentials: 'same-origin',
    });
    this.table.data = await res.json();
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
    }
    .right {
      width: 300px;
      flex-shrink: 1;
    }
  }
</style>
