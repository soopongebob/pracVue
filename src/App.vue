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
      </div>
</template>

<script>
import {computed, ref} from 'vue';
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
        const todoStyle = {
            textDecoration: 'line-through',
            color: 'gray'
        }
        const todos = ref([]);
        
        const getTodos = async () => {
            try{
                const res = await axios.get('http://localhost:3000/todos');
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
        const toggleTodo = (index) => {
            todos.value[index].completed = !todos.value[index].completed;
        }
        const count = ref(1);
        const doubleCount = computed(() => {
            return count.value * 2;
        });
        const deleteTodo = (index) => {
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
