<script setup>
import {reactive, ref, onMounted} from 'vue';
import axios from "axios";

axios.interceptors.request.use( ( config ) => {
  state.error = null;
  return config;
} );

axios.interceptors.response.use( ( response ) => {
  return response;
}, function ( error ) {
  state.error = error.response?.data?.message || error.message;
  return error;
} );

const state = reactive( {
  name: '',
  users: [
    {
      name: 'Radu'
    }, {
      name: 'Radu 2'
    }
  ],
  error: null,
  loading: true,
  saving: false,
} )

async function onSubmit() {
  state.saving = true;
  await axios.post( 'https://api.aws.groza.app/users', {
    name: state.name
  } );
  state.saving = false;
  loadUsers();
}

async function loadUsers() {
  state.loading = true;
  const {data} = await axios.get( 'https://api.aws.groza.app/users' );
  state.users = data || [];
  state.loading = false;
}

onMounted( () => {
  loadUsers();
} );
</script>

<template>
  <div class="grid grid-cols-3 gap-x-6 max-w-6xl items-start mx-auto p-4 border">
    <div>
      <h2 class="font-semibold">Users</h2>
      <p class="italic" v-if="state.loading">Loading users ...</p>
      <ul v-if="!state.loading && state.users.length">
        <li v-for="user in state.users" :key="user.id" class="flex items-center justify-between p-2">
          <span>{{ user.name }}</span>
        </li>
      </ul>
    </div>

    <form action="" @submit.prevent="onSubmit" class="grid gap-x-4 items-center grid-cols-[1fr_auto]">
      <label class="col-span-2">Add user:</label>
      <input class="rounded-md p-2 focus:shadow focus:shadow-cyan-600 transition-all text-gray-700 focus:outline-none"
          type="text" placeholder="Name" v-model="state.name"/>
      <button class="bg-cyan-800 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-md disabled:opacity-30"
          type="submit" :disabled="state.saving">
        Add
      </button>
    </form>

    <div v-if="state.error"
        class="bg-red-100 border border-red-400 text-red-700 p-4 mt-4 rounded relative"
        role="alert">
      <div class="font-bold">Error!</div>
      <span class="block sm:inline">{{ state.error }}</span>
    </div>

  </div>
</template>

<style scoped>
</style>
