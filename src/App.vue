<template>
    <router-view />
      <div class="container">
            <h2>To-Do List</h2>
          <input class="form-control" type="text" v-model="searchText"
                 placeholder="search" @keyup.enter="searchTodo">
          <hr />
          <TodoSimpleForm @add-todo="addTodo" />

          <div v-if="!todos.length">
              nothing to display
          </div>
          <TodoList :todos="todos" @toggle-todo="toggleTodo" @delete-todo="deleteTodo" />
          <hr />
          <nav aria-label="Page navigation example">
              <ul class="pagination">
                  <li v-if="currentPage !== 1" class="page-item">
                      <a class="page-link" href="javascript:void(0)" @click="getTodos(currentPage -1)">Previous</a>
                  </li>
                  <li class="page-item" v-for="page in numberOfPages" :key="page"
                  :class="currentPage === page ? 'active' : ''">
                      <a class="page-link" href="javascript:void(0)" @click="getTodos(page)">{{page}}</a>
                  </li>
                  <li v-if="numberOfPages !== currentPage" class="page-item">
                      <a class="page-link" href="javascript:void(0)" @click="getTodos(currentPage + 1)">Next</a>
                  </li>
              </ul>
          </nav>
      </div>
</template>

<script>
import {computed, ref, watch} from 'vue';
import TodoSimpleForm from './components/TodoSimpleForm.vue';
import TodoList from "@/components/TodoList.vue";
import axios from "axios";
export default {
    components: {
        TodoList,
        TodoSimpleForm
    },
    setup(){
        const todoStyle = {
            textDecoration: 'line-through',
            color: 'gray'
        }
        const todos = ref([]);
        const numberOfTodos = ref(0);
        const limit = 5;
        const currentPage = ref(1);
        const searchText = ref('');
        const numberOfPages = computed(() =>{
            return Math.ceil(numberOfTodos.value / limit);
        });
        const getTodos = async (page = currentPage.value) => {
            currentPage.value = page;
            try{
                const res = await axios.get(
                    `http://localhost:3000/todos?_sort=id&_order=desc&subject_like=${searchText.value}&_page=${page}&_limit=${limit}`);
                numberOfTodos.value = res.headers['x-total-count'];
                todos.value = res.data;
            }catch (e) {
                console.log(e);
            }
        };
        
        getTodos();
        const addTodo = async (todo) =>{
            //db에 todo 저장. id는 자동생성. 비동기 통신 promise
            //응답이 왔을 때 .then()으로 받음
            try{
                await axios.post('http://localhost:3000/todos', {
                subject: todo.subject,
                completed: todo.completed,
                });
                getTodos(1);
            } catch (err) {
                console.log(err)
            }
        };
        const toggleTodo = (index) => {
            todos.value[index].completed = !todos.value[index].completed;
        }
        const count = ref(1);
        const doubleCount = computed(() => {
            return count.value * 2;
        });
        const deleteTodo = async (index) => {
            const id = todos.value[index].id;
            try{
                await axios.delete('http://localhost:3000/todos/' + id);
                
                getTodos(1);
            } catch (err) {
                console.log(err);
            }

            getTodos(1);
        }

        const searchTodo = () => {
            clearTimeout(timeout);
            getTodos(1);
        }
        
        let timeout = null
        watch(searchText, () => {
            clearTimeout(timeout);
            timeout = setTimeout(() => {
                getTodos(1);
            }, 2000);
        })
        // const filteredTodos = computed(() => {
        //     if(searchText.value){
        //         return todos.value.filter(todo => {
        //             return todo.subject.includes(searchText.value);
        //         });
        //     }
        //     return todos.value;
        // })
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
            // filteredTodos,
            numberOfPages,
            currentPage,
            searchTodo,
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
