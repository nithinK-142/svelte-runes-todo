<script lang="ts">
  type Todo = {
    text: string;
    done: boolean;
  };

  type Filters = "all" | "active" | "completed";

  let todos = $state<Todo[]>([]);
  let filter = $state<Filters>("all");
  let filteredTodos = $derived(filterTodos());

  $effect(() => {
    const savedTodos = localStorage.getItem("todos");
    savedTodos && (todos = JSON.parse(savedTodos));
  });

  $effect(() => {
    localStorage.setItem("todos", JSON.stringify(todos));
  });

  function addTodo(event: KeyboardEvent) {
    if (event.key !== "Enter") return;

    const todoElement = event.target as HTMLInputElement;

    // const id = window.crypto.randomUUID(); generates ID

    const text = todoElement.value;
    const done = false;

    todos = [...todos, { text, done }];
    todoElement.value = "";
  }

  function editTodo(event: Event) {
    const inputElement = event.target as HTMLInputElement;

    const index = +inputElement.dataset.index!;
    todos[index].text = inputElement.value;
  }

  function toggleTodo(event: Event) {
    const inputElement = event.target as HTMLInputElement;

    const index = +inputElement.dataset.index!;
    todos[index].done = !todos[index].done;
  }

  function setFilter(newFilter: Filters) {
    filter = newFilter;
  }

  function filterTodos() {
    switch (filter) {
      case "all":
        return todos;
      case "active":
        return todos.filter((todo) => !todo.done);
      case "completed":
        return todos.filter((todo) => todo.done);
    }
  }

  function remaining() {
    return todos.filter((todo) => !todo.done).length;
  }

  $effect(() => {
    console.log(todos);
  });
</script>

<div class="todos">
  <input onkeydown={addTodo} placeholder="Add todo" type="text" id="add-todo" />

  {#each filteredTodos as todo, i}
    <div class:completed={todo.done} class="todo">
      <input oninput={editTodo} value={todo.text} data-index={i} type="text" />
      <input
        onchange={toggleTodo}
        value={todo.text}
        data-index={i}
        checked={todo.done}
        type="checkbox"
      />
    </div>
  {/each}
  <div class="filters">
    <button onclick={() => setFilter("all")}>All</button>
    <button onclick={() => setFilter("active")}>Active</button>
    <button onclick={() => setFilter("completed")}>Completed</button>
  </div>

  <p>{remaining()} remaining.</p>
</div>

<style>
  .todos {
    min-height: 90vh;

    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 1rem;
  }

  .todo {
    position: relative;
    transition: opacity 0.3s;
  }

  .completed {
    opacity: 0.4;
  }

  #add-todo {
    width: 20%;
  }

  input[type="text"] {
    padding: 1rem;
    font-size: 1rem;
  }

  input[type="checkbox"] {
    position: absolute;
    top: 50%;
    translate: 0% -50%;
    right: -25%;
  }
</style>
