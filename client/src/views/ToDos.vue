<template>
  <div>
    <modal
      v-bind:is-showing="isVisible"
      title="Add To Do Item"
      success-button="Add"
      v-on:success="addToDoItem"
      v-on:cancel="cancel"
    >
      <form v-on:submit.prevent="onSubmit">
        <div class="field">
          <label class="label">Add Item</label>
          <div class="control">
            <input class="input" type="text" placeholder="add item" v-model="newItem.title">
          </div>
        </div>
        <div class="field">
          <label class="label">Due Date</label>
          <div class="control">
            <input class="input" type="date" placeholder="date" v-model="newItem.duedate">
          </div>
        </div>
      </form>
    </modal>
    <h1 id="to-do-header">My To-Do List</h1>
    <table>
      <thead>
        <tr>
          <th>Item</th>
          <th>Due Date</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(todo, index) in mytodos" v-bind:key="index" class="to-do-row">
          <td class="option-col">
            <input type="checkbox" name="option{index}">
            {{ todo.title }}
            <br>
          </td>
          <td class="due-date">{{ todo.duedate }}</td>
          <td class="delete">
            <button class="button">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
    <button class="button" v-on:click="showAddModal">Add</button>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { Component, Prop } from "vue-property-decorator";
import Modal from "../components/Modal.vue";
import axios, { AxiosResponse } from "axios";
import { APIConfig } from "../utils/api.utils";

@Component({
  components: {
    Modal
  }
})
export default class ToDos extends Vue {
  @Prop(Boolean) isVisible = false;
  mytodos: ToDo[] = [];

  mounted() {
    axios.get(APIConfig.buildUrl("/todos"), {}).then((response: AxiosResponse<ToDo[]>) => {
      for (var i = 0; i < response.data.length; i++) {
        const item: ToDo = response.data[i]
        console.log(item)
        this.mytodos.push(item)
      }
    }).catch((response: AxiosResponse) => {
      console.log("error getting stuff");
    })
  }

  newItem: ToDo = {
    title: "",
    duedate: undefined,
    completed: false
  };

  showAddModal() {
    this.isVisible = true;
    console.log("Opened")
  }

  hideAddModal() {
    this.isVisible = false;
  }

  addToDoItem() {
    const newTodo: ToDo = {
      title: this.newItem.title,
      duedate: this.newItem.duedate,
      completed: this.newItem.completed
    }
    axios.post(APIConfig.buildUrl("/todos"), {
      title: newTodo.title,
      duedate: newTodo.duedate,
      completed: newTodo.completed
    }).then((response: AxiosResponse<ToDo>) => {
      debugger;
      this.mytodos.push(response.data);
    }).catch((response: AxiosResponse) => {
      console.log(response)
    });
    this.hideAddModal();
  }

  cancel() {
    this.hideAddModal();
  }
}

interface ToDo {
  title: string;
  duedate: Date | undefined;
  completed: boolean;
}
</script>

<style scoped>
table {
  width: 500px;
  margin-left: 30px;
}

th {
  text-align: left;
}

.due-date {
  font: normal;
}

.option-col {
  text-align: left;
}

.delete {
  text-align: center;
}

.delete-button {
  background-color: firebrick;
}

.add-button {
  margin-left: 35px;
  background-color: lightgreen;
}

.cancel-button {
  background-color: lightgray;
}

.add-todo-item {
  text-align: left;
}

#to-do-header {
  margin-bottom: 10px;
}
</style>
