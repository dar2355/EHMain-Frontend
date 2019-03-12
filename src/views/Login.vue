<template>
  <el-col id="loginform" :span="11" @submit.native.prevent>
    <el-form ref="form" :model="form" label-width="120px">
      <el-form-item label="username">
        <el-input v-model="form.username"></el-input>
      </el-form-item>
      <el-form-item label="password">
        <el-input type="password" v-model="form.password"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit">Submit</el-button>
      </el-form-item>
    </el-form>
    <el-alert
      :title="alert.message"
      :type="alert.type"
      :description="alert.description"
      v-if="alert.visible"
      @close="alert.visible = false"
      show-icon>
    </el-alert>
  </el-col>
</template>

<script lang="ts">
import Vue from 'vue';
import { stringify } from 'query-string';

export default Vue.extend({
  name: 'login',
  data: () => {
    return {
      form: {
        username: '',
        password: '',
      },
      alert: {
        visible: false,
        type: 'success',
        message: '',
        description: '',
      }
    };
  },
  methods: {
    async handleResponse(res: Response) {
      const data = await res.json();

      if (data.token) {
        this.alert.type = 'success';
        this.alert.message = `success!`;
        this.alert.description = ``;
        window.sessionStorage.setItem('jwt', `Bearer ${data.token}`);
      } else {
        this.alert.type = 'error';
        this.alert.message = 'error';
        this.alert.description = data.error;
      }
      this.alert.visible = true;
    },
    async onSubmit() {
      const res = await fetch(`http://127.0.0.1:3000/api/login?${stringify(this.form)}`, {
        method: 'POST',
      });
      this.handleResponse(await res);
    },
  },
});
</script>

<style lang="scss">
  #loginform {
    text-align: left;
  }
</style>