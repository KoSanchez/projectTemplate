<template>
    <div class='todos'>
        <table>
            <thead>
                <th/>
                <th> Item </th>
                <th> Due Date </th>
                <th/>
            </thead>
        <!--actual to do list -->
        <!--group the box with todo item-->
            <tbody>
                <tr v-for="(todo, index) in mytodos" v-bind:key="index">
                    <td><input type="checkbox" v-on:change="markDone(index)" v-bind:checked="todo.complete"></td>
                    <td>{{ todo.title }}</td>
                    <td>{{ todo.dueDate || "N/A" }}</td>
                    <td><button class="button" v-on:click="deleteTodoItem(todo.id)"> Delete </button></td>
                </tr>
            </tbody>
        </table>
        <div class="add-new">
            <button class="button" v-on:click="addTodoItem"> Add </button>
            <input id="dateInput" type="date" class="input"/>
            <input id="descInput" type="text" placeholder="ToDo description" class="input/">
        </div>
        
    </div>
</template>

<script lang="ts">
import axios, { AxiosError, AxiosResponse } from "axios";
import Vue from 'vue';
import { APIConfig } from "../utils/api.utils";
import { Component } from "vue-property-decorator";
@Component
export default class ToDos extends Vue {
    idVal: number = 0;
    mytodos: ToDo[] = [
        /*{ name: "Todo one", duedate: undefined},
        { name: "todo two", duedate: undefined, done: true },
        { name: "todo three", duedate: undefined}*/
        { title: "todo one", dueDate: undefined},
        { title: "todo two", dueDate: undefined, complete: true },
        { title: "todo three", dueDate: undefined}
        
    ];
    addTodoItem() {
        const dueDate = (<HTMLInputElement>document.getElementById("dateInput")).value;
        const description = (<HTMLInputElement>document.getElementById("descInput")).value;
        if(description) {
            const todo: ToDo = {title: description, dueDate: dueDate ? new Date(dueDate) : undefined, complete: false};
            axios
                .post(APIConfig.buildUrl("/todos"), todo, { 
                    headers: {
                        token: this.$store.state.userToken
                    }
                })
                .then((response) => {
                    this.mytodos.push(response.data.todo);
                });
        }
    }
    deleteTodoItem(index: number) {
        const todo = this.mytodos[index];
        axios
            .delete(APIConfig.buildUrl("/todos/" + todo.id), {
                headers: {
                    token: this.$store.state.userToken
                }
            })
            .then(() => {
                this.mytodos.splice(index, 1);
            });
    }
    markDone(index: number) {
        const complete = !this.mytodos[index].complete;
        this.mytodos[index].complete = complete;
        const todo = this.mytodos[index];
        axios
            .put(APIConfig.buildUrl("/todos/" + todo.id), { complete: complete }, {
                headers: {
                    token: this.$store.state.userToken
                }
            })
    }
    loadTodos() {
        if (this.$store.state.userToken) {
            axios
                .get(APIConfig.buildUrl("/todos/load"), {
                    headers: {
                        token: this.$store.state.userToken
                    }
                })
                .then((response) => {
                    this.mytodos = response.data;
                });
        }
    }
    mounted() {
        this.loadTodos();

        this.$store.watch(
            (state) => state.userToken,
            (newValue, oldValue) => {
                if (newValue) {
                    this.loadTodos();
                }
            }
        );
    }
    beforeRouteUpdate(to: string, from: string, next: string) {
        if (to == "todos") {
            this.loadTodos();
        }
    }
}
interface ToDo {
    title: string;
    dueDate?: Date;
    complete?: boolean;
    id?: number;
}
</script>

<style scoped>
        /*ul {
          list-style: none;
          position: relative;
        }
        ul.list {
          display: inline-block;
          vertical-align: top;
        }
        .title{
          margin-left: 32px;
        }
        .due-date {
          margin-left: 3px;
        }
        .buttons {
          margin-left: 170px;
        }
        .cancel {
          margin-left: 50px;
        }*/
</style>
