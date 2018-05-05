<template>
  <q-page padding>
      <q-input v-model="text"  @keyup.enter="addTodo" />
      <q-btn v-if="formState === 'add'" label="ADD" @click="createTodo" />
      <q-btn v-else label="UPDATE" @click="updateTodo" />
      <q-btn v-if="formState !== 'add'" label="Cancel" @click="cancelUpdate" />
      <br><br>
      Search
      <q-input v-model="search"  />
      <br>
      <q-list>
        <q-list-header>Todos</q-list-header>
        <q-item v-for="(item, index) in todos" :key="item.id">
            {{ item.text  }}
            <q-btn @click="editTodo(item)" label="Edit" />
            <q-btn @click="deleteTodo(item.id)" label="Delete" />
        </q-item>
        <q-item v-if="loading">
            loading...
        </q-item>
      </q-list>
  </q-page>
</template>

<script>
export default {
  data () {
    return {
      todos: [],
      text: '',
      todo: '',
      formState: 'add',
      loading: false,
      search: ''
    }
  },
  watch: {
    search () {
      console.log('the search bar changed.')
      this.getTodos()
    }
  },
  created () {
    this.getTodos()
  },
  methods: {
    addTodo () {
      this.todos.push({
        id: Math.random(),
        text: this.text,
        done: false
      })
      this.text = ''
    },
    editTodo (todo) {
      this.formState = 'edit'
      this.todo = Object.assign({}, todo)
      this.text = todo.text
    },
    update () {
       let index = this.todos.findIndex(value => value.id === this.todo.id)
       this.todos[index].text = this.text
       this.todos[index].id = this.todo.id
       this.todos[index].done = this.todo.done
       this.cancelUpdate()
    },
    cancelUpdate () {
      this.formState = 'add'
      this.todo = ''
      this.text = ''
    },
    getTodos () {
      this.loading = true
      let search = this.search ? '?search=' + this.search : ''
      this.$axios.get('http://localhost:8000/todos' + search).then(response => {
         this.todos = response.data
         this.loading = false
      })
    },
    createTodo () {
      let formData = {
        text: this.text,
        done: false
      }
      this.$axios.post('http://localhost:8000/todos', formData).then(response => {
          this.getTodos()
          this.cancelUpdate()
      })
    },
    deleteTodo (id) {
      this.$axios.delete('http://localhost:8000/todos/' + id).then(response => {
          this.getTodos()
      })
    },
    updateTodo () {
      let formData = {
        text: this.text,
        done: this.todo.done,
        _method: 'put'
      }
      this.$axios.post('http://localhost:8000/todos/' + this.todo.id, formData).then(response => {
          this.getTodos()
          this.cancelUpdate()
      })
    }
  }
}
</script>

<style>
</style>
