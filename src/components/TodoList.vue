<template>
    <div
            v-for="(todo, index) in todos"
            :key="todo.id"
            class="card mt-2">
        <div class="card-body p-2 d-flex align-items-center">
            <div class="form-check flex-grow-1">
                <input class="form-check-input" :checked="todo.completed" @change="toggleTodo(index)"
                       type="checkbox">
                <label class="form-check-label"
                       :class="{ todo: todo.completed }">{{ todo.subject }}</label>
            </div>
            <button class="btn btn-danger btn-sm"
                    @click="deleteTodo(index)">Delete</button>
        </div>
    </div>
</template>

<script>
export default {
name: "TodoList",
    props: {
      todos: {
          type: Array,
          required: true
      }
    },
    emits: ['toggle-todo', 'delete-todo'],
    setup(props, { emit }){
        const toggleTodo = (index) => {
            emit('toggle-todo', index);
        };
        const deleteTodo = (index) => {
            emit('delete-todo', index);
        }
        return{
            toggleTodo,
            deleteTodo,
        }
    }
}
</script>

<style scoped>

</style>