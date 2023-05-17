<template>
      <div class="container">
            <h2>To-Do List</h2>
          <input class="form-control" type="text" v-model="searchText"
                 placeholder="search">
          <hr />
          <TodoSimpleForm @add-todo="addTodo" />

          <div v-if="!filteredTodos.length">
              nothing to display
          </div>
          <TodoList :todos="filteredTodos" @toggle-todo="toggleTodo" @delete-todo="deleteTodo"/>
          <hr />
          <nav aria-label="Page navigation example">
              <ul class="pagination">
                  <li class="page-item" v-if="currentPage !== 1" >
                      <a class="page-link" @click="getTodos(currentPage - 1)" style="cursor: pointer">
                          pre
                      </a>
                  </li>
                  <li class="page-item" v-for="page in pageCount" :key="page" :class="currentPage === page ? 'active' : ''">
                      <a class="page-link" @click="getTodos(page)" style="cursor: pointer">
                          {{ page }}
                      </a>
                  </li>
                  <li class="page-item" v-if="currentPage !== pageCount" >
                      <a class="page-link" @click="getTodos(currentPage + 1)" style="cursor: pointer">
                          next
                      </a>
                  </li>
              </ul>
          </nav>
      </div>
</template>

<script>
import {computed, ref } from 'vue';
import TodoSimpleForm from './components/TodoSimpleForm.vue';
import TodoList from "@/components/TodoList.vue";
import axios from "axios";
export default {
    props: {
        
    },
    components: {
        TodoList,
        TodoSimpleForm
    },
    setup(){
        const todosCount = ref(0);
        const currentPage = ref(1);
        const limit = 5;
        const pageCount = computed(() => {
            return Math.ceil(todosCount.value / limit)
        });

        
        
        const todoStyle = {
            textDecoration: 'line-through',
            color: 'gray'
        }
        const todos = ref([]);
        
        const getTodos = async (page = currentPage.value) => {
            currentPage.value = page;
            try{
                const res = await axios.get(
                    `http://localhost:3000/todos?_page=${page}&_limit=${limit}`
                );
                todos.value = res.data;
                todosCount.value = res.headers['x-total-count'];
            }catch (e) {
                console.log(e);
            }
        };
        
        getTodos();
        const addTodo = async (todo) =>{
            //db에 todo 저장. id는 자동생성. 비동기 통신 promise
            //응답이 왔을 때 .then()으로 받음
            try{
                const res = await axios.post('http://localhost:3000/todos', {
                subject: todo.subject,
                completed: todo.completed,
                });
                //보낸 응답이 끝나고 이상 없으면 아래 코드 실행
                todos.value.push(res.data);
            } catch (err) {
                console.log(err)
            }
        };
        const toggleTodo = async (index) => {
            try{
                const id = todos.value[index].id;
                const res = await axios.patch('http://localhost:3000/todos/' + id, {
                    completed: !todos.value[index].completed,
                });
                console.log(res);
            } catch (err) {
                console.log(err)
            }
            todos.value[index].completed = !todos.value[index].completed;
        }
        const count = ref(1);
        const doubleCount = computed(() => {
            return count.value * 2;
        });
        const deleteTodo = async (index) => {
            try{
                const id = todos.value[index].id;
                await axios.delete('http://localhost:3000/todos/' + id);
            } catch (err) {
                console.log(err)
            }
            todos.value.splice(index, 1);
        }
        const searchText = ref('');
        const filteredTodos = computed(() => {
            if(searchText.value){
                return todos.value.filter(todo => {
                    return todo.subject.includes(searchText.value);
                });
            }
            return todos.value;
        })
        return {
            todos,
            todoStyle,
            deleteTodo,
            getTodos,
            addTodo,
            toggleTodo,
            count,
            doubleCount,
            searchText,
            filteredTodos,
            pageCount,
            currentPage,
            todosCount,
        };
    }
}
</script>

<style>
  .todo{
      color:gray;
      text-decoration: line-through;
  }
/*#app {*/
/*  font-family: Avenir, Helvetica, Arial, sans-serif;*/
/*  -webkit-font-smoothing: antialiased;*/
/*  -moz-osx-font-smoothing: grayscale;*/
/*  text-align: center;*/
/*  color: #2c3e50;*/
/*  margin-top: 60px;*/
/*}*/
</style>
