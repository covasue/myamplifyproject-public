<template>
  <div id="app">
    <h1>Todo App</h1>
    <input type="text" v-model="name" placeholder="Todo name" />
    <input type="text" v-model="description" placeholder="Todo description" />
    <button v-on:click="createTodo">Create Todo</button>
  </div>
</template>

<script>
import { API } from 'aws-amplify';
import { createTodo } from './graphql/mutations';

export default {
  name: 'app',
  data() {
    return {
      name: '',
      description: ''
    }
  },
  methods: {
    async createTodo() {
      const { name, description } = this;
      if (!name || !description) return;
      const todo = { name, description };
      await API.graphql({
        query: createTodo,
        variables: { input: todo },
      });
      this.name = '';
      this.description = '';
    }
  }
};
</script>