<script setup>
import { computed, onMounted, ref } from "vue";
import { useTodoStore } from "../stores/todo";
import { RouterLink } from "vue-router";
import Loading from "../components/Loading.vue";

const todoStore = useTodoStore();
const todoText = ref("");
const isLoading = ref(false);
const selectedStatus = ref("Pending");
const filterTodoList = computed(() => {
  return todoStore.list.filter((todo) => todo.status === selectedStatus.value);
});

onMounted(async () => {
  isLoading.value = true;
  await todoStore.loadTodos();
  isLoading.value = false;
});

const addTodo = async (todoText) => {
  try {
    isLoading.value = true;
    await todoStore.addTodo(todoText);
    isLoading.value = false;
  } catch (error) {
    console.log("error", error);
  }
};

const editStatus = async (todoData, todoId) => {
  const updateData = {
    name: todoData.name,
    status: todoData.status,
  };
  try {
    isLoading.value = true;
    await todoStore.editTodo(updateData, todoId);
    isLoading.value = false;
  } catch (error) {
    console.log("error", error);
  }
};

const deleteTodo = async (todoId) => {
  try {
    isLoading.value = true;
    await todoStore.removeTodo(todoId);
    await todoStore.loadTodos();
    isLoading.value = false;
  } catch (error) {
    console.log("error", error);
  }
};

const changeStatus = async (event, todoId) => {
  if (event.target.checked) {
    try {
      isLoading.value = true;
      await todoStore.editTodo({ status: "Done" }, todoId);
      await todoStore.loadTodos();
      isLoading.value = false;
    } catch (error) {
      console.log("error", error);
    }
  } else {
    try {
      isLoading.value = true;
      await todoStore.editTodo({ status: "Doing" }, todoId);
      await todoStore.loadTodos();
      isLoading.value = false;
    } catch (error) {
      console.log("error", error);
    }
  }
};

const changeSelectedStatus = async (newStatus) => {
  selectedStatus.value = newStatus;
};
</script>

<template>
  <div>
    <div class="flex gap-2">
      <input
        class="input input-bordered w-full"
        placeholder="Todo..."
        type="text"
        v-model="todoText"
      /><button class="btn btn-primary" @click="addTodo(todoText)">Add</button>
    </div>
    <Loading v-if="isLoading" />

    <div class="flex my-4">
      <div role="tablist" class="tabs tabs-boxed">
        <a
          :key="index"
          role="tab"
          v-for="status in todoStore.statuses"
          :class="status === selectedStatus ? 'tab tab-active' : 'tab'"
          @click="changeSelectedStatus(status)"
        >
          {{ status }}
        </a>
      </div>
    </div>

    <div
      class="flex items-center justify-between mt-2"
      v-for="todo in filterTodoList"
      :key="todo.id"
    >
      <div>
        <input
          class="checkbox"
          type="checkbox"
          :checked="todo.status === 'Done'"
          @change="changeStatus($event, todo.id)"
        />
      </div>
      <div :class="todo.status === 'Done' ? 'line-through' : ''">
        {{ todo.name }}
      </div>
      <div class="flex gap-2">
        <RouterLink :to="{ name: 'todo-edit', params: { id: todo.id } }">
          <button class="btn btn-square btn-outline">
            <font-awesome-icon icon="pen-to-square" />
          </button>
        </RouterLink>

        <button class="btn btn-square btn-outline" @click="deleteTodo(todo.id)">
          <font-awesome-icon icon="trash" />
        </button>
      </div>
    </div>
  </div>
</template>
