<script setup>
import { computed, ref, watch } from 'vue';

const STORAGE_KEY = 'vuetify-todos';
const todoStorage = {
  fetch() {
    const todos = JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
    todos.forEach((todo, index) => todo.id = index);
    todoStorage.uid = todos.length;
    return todos;
  },
  save(todos) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
  }
}

const filters = {
  all(todos) {
    return todos;
  },
  active(todos) {
    return todos.filter(todo => !todo.completed);
  },
  completed(todos) {
    return todos.filter(todo => todo.completed);
  }
}

const todos = ref(todoStorage.fetch());
const newTodo = ref('');
const editedTodo = ref({});
const tab = ref('all');

const filteredTodos = computed(() => filters[tab.value](todos.value));
const activeTodosNum = computed(() => filters.active(todos.value).length);

watch(todos, () => {
  todoStorage.save(todos.value);
}, { deep: true })

function addTodo() {
  const todoTitle = newTodo.value.trim();
  if(!todoTitle) return;
  todos.value.push({
    title: todoTitle,
    id: todoStorage.uid++,
    completed: false
  });
  newTodo.value = '';
}

function changeTodoStatus(todo) {
  todo.completed = !todo.completed;
}

function editTodo() {
  const newTitle = editedTodo.value.title.trim()
  if(!newTitle) return
  const index = todos.value.findIndex(todo => todo.id === editedTodo.value.id);
  todos.value[index].title = newTitle;
  editedTodo.value = {};
}

function removeTodo(todo) {
  const index = todos.value.indexOf(todo);
  todos.value.splice(index, 1);
}

function removeCompleted() {
  todos.value = filters.active(todos.value);
}

function removeAll() {
  todos.value = [];
}
</script>

<template>
  <v-text-field
    append-inner-icon="mdi-plus"
    label="新增代辦事項"
    variant="underlined"
    v-model="newTodo"
    @click:append-inner="addTodo"
    @keyup.enter="addTodo"
  >
  </v-text-field>
  <v-card>
    <v-tabs
      v-model="tab"
      bg-color="primary"
      align-tabs="center"
    >
      <v-tab value="all">全部</v-tab>
      <v-tab value="active">未完成</v-tab>
      <v-tab value="completed">已完成</v-tab>
    </v-tabs>

    <template v-if="todos.length">
      <v-list>
        <v-list-item v-for="todo in filteredTodos" :key="todo.id" class="cursor-pointer">
          <v-list-item-title v-if="editedTodo?.id === todo.id">
            <v-text-field
              append-inner-icon="mdi-pencil"
              variant="underlined"
              v-model="editedTodo.title"
              @click:append-inner="editTodo"
              @keyup.enter="editTodo"
              @keyup.esc="editedTodo = null"
            >
            </v-text-field>
          </v-list-item-title>
          <v-list-item-title class="d-flex align-items-center todo-item" @click="changeTodoStatus(todo)" v-else>
              <span class="my-auto" :class="{'text-decoration-line-through' : todo.completed}" id="todoItem">
              <v-icon class="me-2">{{ todo.completed ? 'mdi-checkbox-marked': 'mdi-checkbox-blank-outline' }}</v-icon>
                {{ todo.title }}
              </span>
              <v-btn
                icon
                variant="plain"
                color="grey"
                class="rounded-circle ms-auto"
                @click.stop="editedTodo = { ...todo }"
              >
                <v-icon>mdi-pencil</v-icon>
              </v-btn>
              <v-btn
                icon
                variant="plain"
                color="grey"
                class="rounded-circle"
                @click.stop="removeTodo(todo)"
              >
                <v-icon>mdi-delete</v-icon>
              </v-btn>
          </v-list-item-title>
        </v-list-item>
      </v-list>

      <v-divider class="mx-4 mb-1"></v-divider>
      <div class="d-sm-flex px-4 py-2">
        <span class="my-auto text-grey-darken-1 text-body-2 d-block">尚有 {{ activeTodosNum }} 件未完成事項</span>
        <v-btn class="text-end ms-sm-auto bg-grey-lighten-2 me-2 my-2 my-sm-0" variant="plain" @click="removeCompleted">移除已完成項目</v-btn>
        <v-btn class="text-end bg-grey-lighten-2" variant="plain" @click="removeAll">移除所有項目</v-btn>
      </div>
    </template>

    <p class="px-4 py-10" v-else>目前沒有代辦事項，新增一個吧！</p>
  </v-card>
</template>

<style scoped>
.v-list {
  min-height: 200px;
}
.todo-item:hover {
  background-color: #FAFAFA;
}
</style>