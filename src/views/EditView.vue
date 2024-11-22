<script setup>
import { useTodoStore } from "@/stores/todo";
import { onMounted, ref } from "vue";
import { useRoute, RouterLink } from "vue-router";

const todoStore = useTodoStore();
const route = useRoute();
const todoId = ref(-1);
const isLoaded = ref(false);

onMounted(async () => {
  try {
    await todoStore.loadTodo(route.params.id);
    isLoaded.value = true;
    todoId.value = parseInt(route.params.id);
  } catch (error) {
    console.log("error", error);
  }
});

const editTodo = async (selectedTodo) => {
  const bodyData = {
    name: selectedTodo.name,
    status: selectedTodo.status,
  };
  try {
    await todoStore.editTodo(bodyData, todoId.value);
    alert("Edit Complete");
  } catch (error) {
    console.log("error", error);
  }
};
</script>

<template>
  <div v-if="isLoaded">
    Edit id {{ todoId }}
    <div>Name: <input type="text" v-model="todoStore.selectedTodo.name" /></div>
    <div>
      Status:
      <select v-model="todoStore.selectedTodo.status">
        <option>Select Status</option>
        <option v-for="status in todoStore.statuses" :value="status">
          {{ status }}
        </option>
      </select>
      <div>
        <button @click="editTodo(todoStore.selectedTodo)">Edit</button
        ><RouterLink :to="{ name: 'todo-list' }"
          ><button>Back</button></RouterLink
        >
      </div>
    </div>
  </div>
  <div v-else><h2>Loading</h2></div>
</template>
