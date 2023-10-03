<template>
  <todo-list />
</template>

<script lang="ts">
import { Options, Vue } from 'vue-class-component';
import TodoList from './TodoList.vue';

// This Application belongs to Chinberdiyev Alisher, and is done for Olcha.uz testing task

@Options({
  components: {
    TodoList
  },
})
export default class App extends Vue { }
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
