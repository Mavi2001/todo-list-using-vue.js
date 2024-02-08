<template>
  <div id="app">
    <h1 class="title">HARSH MAVI'S TODO LIST</h1>
    <div class="input-container">
      <input
        type="text"
        v-model="newTask"
        placeholder="Add new task"
        class="input-field"
      />
      <button @click="addTask" class="add-button">Add Task</button>
    </div>

    <!-- Open Tasks -->
    <h2 class="section-title">Open Tasks</h2>
    <ul v-if="openTasks.length > 0">
      <li v-for="task in openTasks" :key="task.id" class="task-item">
        <span>{{ task.name }}</span>
        <button @click="completeTask(task)" class="complete-button">
          Complete
        </button>
      </li>
    </ul>

    <!-- Completed Tasks -->
    <h2 class="section-title">Completed Tasks</h2>
    <ul v-if="completedTasks.length > 0">
      <li v-for="task in completedTasks" :key="task.id" class="task-item">
        <input
          type="checkbox"
          v-model="selectedCompletedTasks"
          :value="task.id"
          class="checkbox"
        />
        <span class="completed">{{ task.name }}</span>
      </li>
    </ul>
    <button
      @click="deleteSelectedCompletedTasks"
      v-if="selectedCompletedTasks.length > 0"
      class="delete-button"
    >
      Delete Selected
    </button>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "App",
  data() {
    return {
      newTask: "",
      tasks: [],
      selectedCompletedTasks: [],
    };
  },
  computed: {
    openTasks() {
      return this.tasks.filter((task) => !task.completed);
    },
    completedTasks() {
      return this.tasks.filter((task) => task.completed);
    },
  },
  methods: {
    fetchTasks() {
      axios
        .get("http://localhost:3000/tasks")
        .then((response) => {
          this.tasks = response.data;
        })
        .catch((error) => {
          console.error("Error fetching tasks:", error);
        });
    },
    addTask() {
      const newTask = {
        name: this.newTask,
        completed: false,
      };

      axios
        .post("http://localhost:3000/tasks", newTask)
        .then((response) => {
          this.tasks.push(response.data);
          this.newTask = "";
        })
        .catch((error) => {
          console.error("Error adding task:", error);
        });
    },
    completeTask(task) {
      task.completed = true;
      axios
        .put(`http://localhost:3000/tasks/${task.id}`, task)
        .then(() => {
          // Optionally, you can remove the task from the open tasks list
          // this.tasks = this.tasks.filter(t => t.id !== task.id);
        })
        .catch((error) => {
          console.error("Error completing task:", error);
        });
    },
    deleteCompletedTask(taskId) {
      axios
        .delete(`http://localhost:3000/tasks/${taskId}`)
        .then(() => {
          this.tasks = this.tasks.filter((task) => task.id !== taskId);
        })
        .catch((error) => {
          console.error(
            `Error deleting completed task with ID ${taskId}:`,
            error
          );
        });
    },
    deleteSelectedCompletedTasks() {
      this.selectedCompletedTasks.forEach((id) => {
        axios
          .delete(`http://localhost:3000/tasks/${id}`)
          .then(() => {
            this.tasks = this.tasks.filter((task) => task.id !== id);
          })
          .catch((error) => {
            console.error(`Error deleting task with ID ${id}:`, error);
          });
      });
      this.selectedCompletedTasks = [];
    },
  },
  mounted() {
    this.fetchTasks();
  },
};
</script>

<style scoped>
#app {
  font-family: Arial, sans-serif;
  margin: 0 auto;
  padding: 20px;
  max-width: 600px;
}

.title {
  text-align: center;
  margin-bottom: 20px;
  color: #333; /* Dark grey color */
}

.input-container {
  display: flex;
  margin-bottom: 20px;
}

.input-field {
  flex: 1;
  padding: 8px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-right: 10px;
}

.add-button {
  padding: 8px 16px;
  font-size: 16px;
  background-color: #007bff; /* Blue color */
  color: #fff; /* White text color */
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.complete-button {
  padding: 6px 12px;
  font-size: 14px;
  background-color: #28a745; /* Green color */
  color: #fff; /* White text color */
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.delete-button {
  padding: 6px 12px;
  font-size: 14px;
  background-color: #dc3545; /* Red color */
  color: #fff; /* White text color */
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.section-title {
  font-size: 20px;
  margin-top: 30px;
  margin-bottom: 10px;
}

.task-item {
  margin-bottom: 10px;
}

.completed {
  text-decoration: line-through;
}

.checkbox {
  margin-right: 10px;
}
</style>
