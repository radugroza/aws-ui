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
  return Promise.reject( error );
} );

const state = reactive( {
  name: '',
  email: '',
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
  try {
    await axios.put( 'https://api.aws.groza.app/users', {
      name: state.name,
      email: state.email,
    } );
    loadUsers();
    state.name = '';
    state.email = '';
  } catch ( e ) {
    state.error = e.response?.data?.message || e.message
  }
  state.saving = false;
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
  <div class="grid grid-cols-3 gap-x-6 max-w-6xl items-start mx-auto p-4 border rounded-lg">
    <div>
      <h2 class="font-semibold">Users</h2>
      <p class="italic" v-if="state.loading">Loading users ...</p>
      <ul v-if="!state.loading && state.users.length" class="bg-gray-600 flex flex-col items-stretch gap-y-px">
        <li v-for="user in state.users" :key="user.id" class="bg-gray-800 py-1">
          <span>{{ user.user_name }}</span>
        </li>
      </ul>
    </div>

    <section>
      <h2 class="text-lg">Add user:</h2>
      <form action="" @submit.prevent="onSubmit" class="flex flex-col gap-y-2 items-stretch">
        <input
            class="rounded-md p-2 border border-cyan-500 focus:shadow focus:shadow-cyan-600 transition-all text-gray-700 focus:outline-none"
            type="text" placeholder="Name" v-model="state.name"/>
        <input
            class="rounded-md p-2 border border-cyan-500 focus:shadow focus:shadow-cyan-600 transition-all text-gray-700 focus:outline-none"
            type="email" placeholder="Email" v-model="state.email"/>
        <button class="bg-cyan-800 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-md disabled:opacity-30"
            type="submit" :disabled="state.saving">
          Add
        </button>
      </form>
    </section>

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
