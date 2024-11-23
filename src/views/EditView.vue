<script setup>
import LoadingVue from "@/components/Loading.vue";
import { useTodoStore } from "@/stores/todo";
import { onMounted, ref } from "vue";
import { useRoute, RouterLink } from "vue-router";

const todoStore = useTodoStore();
const route = useRoute();
const todoId = ref(-1);
const isLoaded = ref(false);
const isUpdated = ref(false);

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
    isLoaded.value = false;
    await todoStore.editTodo(bodyData, todoId.value);
    isLoaded.value = true;
    isUpdated.value = true;
    setTimeout(() => {
      isUpdated.value = false;
    }, 3000);
  } catch (error) {
    console.log("error", error);
  }
};
</script>

<template>
  <div class="min-h-screen flex flex-col" v-if="isLoaded">
    <div class="w-1/2 my-auto mx-auto">
      <div class="toast toast-bottom toast-center">
        <div v-if="isUpdated" class="alert alert-success text-center">
          <span>Edit successfully.</span>
        </div>
      </div>
      <div class="flex items-center align-middle justify-between">
        <RouterLink :to="{ name: 'todo-list' }">
          <button class="badge badge-secondary inline-block">
            <font-awesome-icon :icon="['fas', 'chevron-left']" />Back
          </button>
        </RouterLink>

        <div>
          Edit Id:
          <div class="badge badge-primary">{{ todoId }}</div>
        </div>
      </div>

      <div>
        <label class="form-control w-full">
          <div class="label">
            <span class="label-text">Name:</span>
          </div>
          <input
            type="text"
            placeholder="Type name todo"
            class="input input-bordered w-full"
            v-model="todoStore.selectedTodo.name"
          />
        </label>

        <label class="form-control w-full">
          <div class="label">
            <span class="label-text">Pick the best fantasy franchise</span>
          </div>
          <select
            class="select select-bordered"
            v-model="todoStore.selectedTodo.status"
          >
            <option disabled selected>Select Status</option>
            <option
              v-for="(status, index) in todoStore.statuses"
              :value="status"
              :key="index"
            >
              {{ status }}
            </option>
          </select>
        </label>

        <div class="flex">
          <button
            class="btn btn-primary w-full mt-4"
            @click="editTodo(todoStore.selectedTodo)"
          >
            Edit
          </button>
        </div>
      </div>
    </div>
  </div>
  <div v-else><LoadingVue /></div>
</template>
