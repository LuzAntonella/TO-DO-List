<template>
  <div id="app">
    <div class="cont">
      <h1>TO DO LIST</h1>
      <div class="add-task">
        <input
          type="text"
          v-model="name"
          placeholder=" Elemento de la lista"
          class="task-input"
        />
        <button
          v-on:click="createTodo"
          type="submit"
          value=""
          class="submit-task"
          title="Add Task"
        />
      </div>
      <ul class="task-list">
        <div v-for="item in pendientes" :key="item.id">
          <li class="task-list-item">
            <label class="task-list-item-label">
              <input type="checkbox" id="" @change="check(item.id)" />
              <span>{{ item.name }}</span>
            </label>
            <span
              v-on:click="deleted(item.id)"
              class="delete-btn"
              title="Delete Task"
            ></span>
          </li>
        </div>
      </ul>
      <br />
      <hr />
      <ul class="task-complet">
        <li>
          <input type="checkbox" name="list" id="nivel1-1" /><label
            class="desplegable"
            for="nivel1-1"
            >Elementos completados</label
          >
          <ul class="interior">
            <div v-for="item in completados" :key="item.id">
              <li class="task-list-item-completados">
                <label class="task-list-item-label">
                  <input
                    type="checkbox"
                    id="completados"
                    @change="check(item.id)"
                  />
                  <span>{{ item.name }}</span>
                </label>
                <span
                  v-on:click="deleted(item.id)"
                  class="delete-btn"
                  title="Delete Task"
                ></span>
              </li>
            </div>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { API, graphqlOperation } from "aws-amplify";
import { createTodo, updateTodo, deleteTodo } from "./graphql/mutations";
import { listTodos } from "./graphql/queries";

export default {
  name: "App",
  data() {
    return {
      name: "",
      pendientes: [],
      completados: [],
      id: "",
    };
  },
  async created() {
    this.getPendientes();
    this.getCompletados();
  },
  methods: {
    deleted(id) {
      this.id = id;
      this.deleteTodo();
    },
    check: function (id) {
      this.id = id;
      this.updateTodo();
    },
    async createTodo() {
      const { name } = this;
      const description = 0;
      if (!name) return;
      const todo = { name, description };
      await API.graphql({
        query: createTodo,
        variables: { input: todo },
      });
      this.name = "";
      this.getPendientes();
    },

    async getPendientes() {
      let filter = {
        description: {
          eq: 0, // filter priority = 1
        },
      };
      const todos = await API.graphql({
        query: listTodos,
        variables: { filter: filter },
      });
      this.pendientes = todos.data.listTodos.items;
    },
    async getCompletados() {
      let filter = {
        description: {
          eq: 1, // filter priority = 1
        },
      };
      const todos = await API.graphql({
        query: listTodos,
        variables: { filter: filter },
      });
      this.completados = todos.data.listTodos.items;
    },
    async deleteTodo() {
      await API.graphql(
        graphqlOperation(deleteTodo, { input: { id: this.id } })
      );
      this.getPendientes();
    },
    async updateTodo() {
      await API.graphql(
        graphqlOperation(updateTodo, {
          input: { id: this.id, description: 1 },
        })
      );
      this.getPendientes();
      this.getCompletados();
    },
  },
};
</script>

<style>
#app {
  max-width: 480px;
  width: 100%;
  max-height: 100%;
  background-color: rgb(255, 255, 255);
  padding: 25px;
  border-radius: 25px;
  /*overflow: auto;*/
  /*color: #222;*/
}
@import "./assets/css/styles.css";
</style>
