<template>
  <div id="app">
    <authenticator>
      <template v-slot="{ user, signOut }">
        <h1>Hello {{ user.username }}!</h1>
        <button @click="signOut">Sign Out</button>
        <h1>Todo App</h1>
        <input type="text" v-model="name" placeholder="Todo name" />
        <input type="text" v-model="description" placeholder="Todo description" />
        <button v-on:click="clickCreateTodo">Create Todo</button>
        <div v-for="item in todos" :key="item.id">
          <h3>{{ item.name }}</h3>
          <p>{{ item.description }}</p>
        </div>
      </template>
    </authenticator>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { API } from 'aws-amplify';
import { createTodo } from './graphql/mutations';
import { listTodos } from './graphql/queries';
import { onCreateTodo } from './graphql/subscriptions';
import { Authenticator } from '@aws-amplify/ui-vue';
import '@aws-amplify/ui-vue/styles.css';
const name = ref(''),
  description = ref(''),
  todos = ref([]);
const clickCreateTodo = async () => {
  if (!name.value || !description.value) return;
  const todo = { name: name.value, description: description.value };
  await API.graphql({
    query: createTodo,
    variables: { input: todo },
  });
  name.value = '';
  description.value = '';
};
const getTodos = async () => {
  const todosData = await API.graphql({
    query: listTodos
  });
  todos.value = todosData.data.listTodos.items;
};
const subscribe = () => {
  API.graphql({ query: onCreateTodo })
    .subscribe({
      next: (eventData) => {
        let todo = eventData.value.data.onCreateTodo;
        if (this.todos.some(item => item.name === todo.name)) return; // remove duplications
        this.todos = [...this.todos, todo];
      }
    });
};
getTodos();
subscribe();
</script>