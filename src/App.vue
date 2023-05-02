<script setup>
import { ref, onMounted, computed, watch } from "vue";

const todos = ref([]);
const name = ref("");
const editName = ref("");
const sortOrder = ref("desc");
const isEdit = ref(null);

const input_content = ref("");
const input_category = ref(null);

const todos_asc = computed(() => {
  const sortedArray = todos.value.sort((a, b) => {
    return sortOrder.value === "desc"
      ? b.createdAt - a.createdAt
      : a.createdAt - b.createdAt;
  });
  return sortedArray;
});

const addTodo = () => {
  if (input_content.value.trim() === "" || input_category.value === null) {
    return;
  }
  todos.value.push({
    content: input_content.value,
    category: input_category.value,
    done: false,
    createdAt: new Date().getTime(),
  });
};

const removeTodo = todo => {
  todos.value = todos.value.filter((t) => t !== todo);
};

const toggleSortOrder = () => {
  sortOrder.value = sortOrder.value === "desc" ? "asc" : "desc";
};

const editTodo = todo => {
  isEdit.value = (isEdit.value === todo.createdAt) ? null : todo.createdAt;
}

watch(
  todos,
  (newVal) => {
    localStorage.setItem("todos", JSON.stringify(newVal));
  },
  { deep: true }
);

watch(name, (newVal) => {
  localStorage.setItem("name", newVal);
});

onMounted(() => {
  name.value = localStorage.getItem("name") || null;
  todos.value = JSON.parse(localStorage.getItem("todos")) || [];
});
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        What's up, <input type="text" placeholder="Name here" v-model="name" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>CREATE A TODO</h3>

      <form @submit.prevent="addTodo">
        <h4>What's on your todo list ?</h4>
        <input
          type="text"
          placeholder="e.g. make a video"
          v-model="input_content"
        />
        <h4>Pick a category</h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              value="business"
              v-model="input_category"
            />
            <span class="bubble business"></span>
            <div>Business</div>
          </label>
          <label>
            <input
              type="radio"
              name="category"
              value="personal"
              v-model="input_category"
            />
            <span class="bubble personal"></span>
            <div>Personal</div>
          </label>
          <input type="submit" value="Add todo" />
        </div>
      </form>
    </section>

    <section class="todo-list">
      <div class="todo-head">
        <h3>TODO LIST</h3>
        <div class="actions">
          <button class="change" @click="toggleSortOrder">
            {{ sortOrder === "desc" ? "Sort Ascending" : "Sort Descending" }}
          </button>
        </div>
      </div>
      <div class="list">
        <div
          v-for="todo in todos_asc"
          :key="todo.createdAt"
          :class="`todo-item ${todo.done && 'done'}`"
        >
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span :class="`bubble ${todo.category}`"></span>
          </label>

          <div class="todo-content">
              <h3 style="margin-bottom: 0" v-if="isEdit !== todo.createdAt">{{ todo.content }}</h3>
              <input class="todo-content-inp" style="border-bottom: 1px solid grey" type="text" v-else v-model="todo.content" />
          </div>

          <div class="actions">
            <button class="edit" @click="editTodo(todo)">{{ isEdit === todo.createdAt ? 'Saved' : 'Edit' }}</button>
          </div>

          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

