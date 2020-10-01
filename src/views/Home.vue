<template>
    <div class="home">
        <h1>Create Todo</h1>

        <form @submit.prevent="createTodo" class="new-todo__form">
            <label><strong>New Todo</strong></label>
            <textarea rows="6" v-model="state.newTodoContent"></textarea>
            <button class="new-todo__form-button">Create</button>
        </form>

        <div v-if="!state.loadingData && state.todos">
            <h1>My Todo List ({{ state.todos.length }})</h1>

            <div class="home__todos-wrapper">
                <div class="home__todos-item" v-for="todo in todos" :key="todo.id">
                    <h2 v-if="!todo.checked" class="home__todo-item-clickable" @click="toggleCheckTodo(todo)">
                        &#9900;
                    </h2>
                    <h2 v-if="todo.checked" class="home__todo-item-clickable" @click="toggleCheckTodo(todo)">
                        &check;
                    </h2>
                    <h2 class="home__todo-item-clickable" @click="toggleCheckTodo(todo)">{{ todo.text }}</h2>
                    <h2 class="home__todo-remove home__todo-item-clickable" @click="deleteTodo(todo)">&cross;</h2>
                </div>
            </div>
        </div>

        <div v-if="state.loadingData">
            <h1>Loading Todos...</h1>
        </div>
    </div>
</template>

<!-- @ is an alias to /src -->
<script>
import { reactive, computed, watchEffect } from 'vue';
import axios from 'axios';

export default {
    name: 'Home',
    components: {},
    props: {},
    setup(props, ctx) {
        const state = reactive({
            loadingData: true,
            todos: null,
            newTodoContent: '',
        });

        // API call to get all todos
        axios
            .get('https://c50dqyosg0.execute-api.us-east-1.amazonaws.com/test/todos')
            .then(function(response) {
                // handle success
                state.todos = response.data;
                state.loadingData = false;
            })
            .catch(function(error) {
                // handle error
                console.log(error);
            });

        const todos = computed(() => state.todos);

        // FUNCTIONS

        function getTodos() {
            state.loadingData = true;
            axios
                .get('https://c50dqyosg0.execute-api.us-east-1.amazonaws.com/test/todos')
                .then(function(response) {
                    // handle success
                    state.todos = response.data;
                    state.loadingData = false;
                })
                .catch(function(error) {
                    // handle error
                    console.log(error);
                });
        }

        function createTodo() {
            if (state.newTodoContent) {
                state.loadingData = true;
                axios
                    .post('https://c50dqyosg0.execute-api.us-east-1.amazonaws.com/test/todos', {
                        text: state.newTodoContent,
                    })
                    .then(function(response) {
                        // handle success
                        state.newTodoContent = '';
                        getTodos();
                        state.loadingData = false;
                    })
                    .catch(function(error) {
                        // handle error
                        console.log(error);
                    });
            }
        }

        function toggleCheckTodo(todo) {
            state.loadingData = true;
            axios
                .put(`https://c50dqyosg0.execute-api.us-east-1.amazonaws.com/test/todos/${todo.id}`, {
                    text: todo.text,
                    checked: !todo.checked,
                })
                .then(function(response) {
                    // handle success
                    todo.checked = !todo.checked;
                    state.loadingData = false;
                })
                .catch(function(error) {
                    // handle error
                    console.log(error);
                });
        }

        function deleteTodo(todo) {
            state.loadingData = true;
            axios
                .delete(`https://c50dqyosg0.execute-api.us-east-1.amazonaws.com/test/todos/${todo.id}`)
                .then(function(response) {
                    // handle success
                    getTodos();
                    state.loadingData = false;
                })
                .catch(function(error) {
                    // handle error
                    console.log(error);
                });
        }

        return {
            state,
            todos,
            createTodo,
            toggleCheckTodo,
            deleteTodo,
        };
    },
};
</script>

<style lang="scss" scoped>
.home {
    display: flex;
    flex-direction: column;
    align-items: center;

    .new-todo__form {
        display: flex;
        flex-direction: column;
        text-align: center;
        background-color: white;
        padding: 40px;
        width: 400px;
        margin-bottom: 20px;

        .new-todo__form-button {
            margin: 0 auto;
            padding: 10px;
            background-color: #42b983;
            color: white;
            border-radius: 5px;
            border: none;
            width: max-content;
            margin-top: 10px;
            cursor: pointer;
        }
    }

    .home__todos-wrapper {
        margin-top: 10px;

        .home__todos-item {
            background-color: white;
            margin: 10px 0px;
            padding: 10px 40px;
            display: flex;
            align-content: center;
            color: #42b983;

            .home__todo-item-clickable {
                margin: 0 20px;
                cursor: pointer;
            }

            .home__todo-remove {
                color: red;
                cursor: pointer;
                margin-left: auto;
            }
        }
    }
}
</style>
