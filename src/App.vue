<template>
  <div class="app-container">
    <div class="todo-container">
      <!-- Header Section -->
      <header class="header">
        <h1 class="title">✨ Task Manager</h1>
        <p class="subtitle">Kelola tugas harianmu dengan mudah</p>
      </header>

      <!-- Input Section -->
      <div class="input-section">
        <div class="input-container" :class="{ 'input-error': showError }">
          <input
            type="text"
            v-model.trim="newTask"
            placeholder="Tambahkan tugas baru..."
            @keyup.enter="addTask"
            class="task-input"
          />
          <button @click="addTask" class="add-button" :disabled="!newTask">
            <span class="button-text">Tambah</span>
            <span class="button-icon">+</span>
          </button>
        </div>
        <p v-if="showError" class="error-message">
          ⚠️ Tugas tidak boleh kosong!
        </p>
      </div>

      <!-- Stats Section -->
      <div class="stats-section" v-if="tasks.length > 0">
        <div class="stat-card">
          <span class="stat-label">Total Tugas</span>
          <span class="stat-value">{{ tasks.length }}</span>
        </div>
        <div class="stat-card">
          <span class="stat-label">Selesai</span>
          <span class="stat-value">{{ completedTasks }}</span>
        </div>
        <div class="stat-card">
          <span class="stat-label">Progres</span>
          <span class="stat-value">{{ progressPercentage }}%</span>
        </div>
      </div>

      <!-- Tasks List -->
      <div class="tasks-container" v-if="tasks.length > 0">
        <div class="tasks-header">
          <h2 class="section-title">Daftar Tugas</h2>
          <button 
            v-if="completedTasks > 0"
            @click="clearCompleted" 
            class="clear-button"
          >
            Hapus yang selesai
          </button>
        </div>

        <transition-group name="task" tag="ul" class="tasks-list">
          <TaskItem
            v-for="(task, index) in tasks"
            :key="task.id"
            :task="task"
            :index="index"
            @toggle-task="toggleTask"
            @delete-task="deleteTask"
          />
        </transition-group>
      </div>

      <!-- Empty State -->
      <div v-else class="empty-state">
        <span class="empty-icon">📝</span>
        <p class="empty-text">Belum ada tugas. Yuk, mulai tambahkan!</p>
      </div>
    </div>
  </div>
</template>

<script>
import Swal from 'sweetalert2';
import TaskItem from './components/TaskItem.vue';

export default {
  name: 'App',
  components: {
    TaskItem
  },

  data() {
    return {
      newTask: "",
      tasks: [],
      showError: false
    };
  },

  computed: {
    completedTasks() {
      return this.tasks.filter(task => task.done).length;
    },
    progressPercentage() {
      return this.tasks.length 
        ? Math.round((this.completedTasks / this.tasks.length) * 100) 
        : 0;
    }
  },

  watch: {
    tasks: {
      handler: 'saveTasks',
      deep: true
    }
  },

  mounted() {
    this.loadTasks();
  },

  methods: {
    addTask() {
      if (this.newTask.trim() === "") {
        this.showError = true;

        // Tampilkan alert jika task tidak valid
        Swal.fire({
          icon: 'error',
          title: 'Oops...',
          text: 'Task cannot be empty!',
        });

        return;
      }

      this.showError = false;
      const newTask = {
        id: Date.now(),
        text: this.newTask,
        done: false,
        createdAt: new Date()
      };

      this.tasks.unshift(newTask);
      this.newTask = "";

      // Tampilkan alert sukses saat task ditambahkan
      Swal.fire({
        icon: 'success',
        title: 'Task Added',
        text: 'Your task has been added successfully!',
      });
    },

    toggleTask(index) {
      if (this.tasks[index]) {
        this.tasks[index].done = !this.tasks[index].done;
      }
    },

    deleteTask(index) {
      this.tasks.splice(index, 1);
    },

    clearCompleted() {
      this.tasks = this.tasks.filter(task => !task.done);
    },

    saveTasks() {
      try {
        localStorage.setItem("tasks", JSON.stringify(this.tasks));
      } catch (e) {
        console.error('Error saving tasks:', e);
      }
    },

    loadTasks() {
      try {
        const savedTasks = localStorage.getItem("tasks");
        if (savedTasks) {
          this.tasks = JSON.parse(savedTasks);
        }
      } catch (e) {
        console.error('Error loading tasks:', e);
        this.tasks = [];
      }
    }
  }
};
</script>


<style scoped>
.app-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 2rem 1rem;
}

.todo-container {
  max-width: 800px;
  margin: 0 auto;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 2rem;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.header {
  text-align: center;
  margin-bottom: 2rem;
}

.title {
  color: white;
  font-size: 2.5rem;
  font-weight: 700;
  margin-bottom: 0.5rem;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.subtitle {
  color: rgba(255, 255, 255, 0.8);
  font-size: 1.1rem;
}

.input-section {
  margin-bottom: 2rem;
}

.input-container {
  display: flex;
  gap: 1rem;
  background: rgba(255, 255, 255, 0.15);
  padding: 0.5rem;
  border-radius: 12px;
  transition: all 0.3s ease;
}

.input-container:focus-within {
  background: rgba(255, 255, 255, 0.2);
}

.task-input {
  flex: 1;
  background: transparent;
  border: none;
  padding: 0.8rem 1rem;
  color: white;
  font-size: 1rem;
}

.task-input::placeholder {
  color: rgba(255, 255, 255, 0.6);
}

.task-input:focus {
  outline: none;
}

.add-button {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  background: #42b883;
  color: white;
  border: none;
  padding: 0.8rem 1.5rem;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.add-button:hover:not(:disabled) {
  background: #3aa876;
  transform: translateY(-1px);
}

.add-button:disabled {
  background: rgba(255, 255, 255, 0.2);
  cursor: not-allowed;
}

.stats-section {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1rem;
  margin-bottom: 2rem;
}

.stat-card {
  background: rgba(255, 255, 255, 0.1);
  padding: 1rem;
  border-radius: 12px;
  text-align: center;
  transition: transform 0.3s ease;
}

.stat-card:hover {
  transform: translateY(-2px);
}

.stat-label {
  display: block;
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9rem;
  margin-bottom: 0.5rem;
}

.stat-value {
  color: white;
  font-size: 1.5rem;
  font-weight: 700;
}

.tasks-container {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 16px;
  padding: 1.5rem;
}

.tasks-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.section-title {
  color: white;
  font-size: 1.2rem;
  font-weight: 600;
}

.clear-button {
  background: rgba(255, 255, 255, 0.1);
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.clear-button:hover {
  background: rgba(255, 255, 255, 0.2);
}

.tasks-list {
  list-style: none;
  padding: 0;
}

.empty-state {
  text-align: center;
  padding: 3rem 0;
  color: rgba(255, 255, 255, 0.8);
}

.empty-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
  display: block;
}

.empty-text {
  font-size: 1.1rem;
}

.error-message {
  color: #ff6b6b;
  margin-top: 0.5rem;
  font-size: 0.9rem;
}

/* Animations */
.task-enter-active,
.task-leave-active {
  transition: all 0.5s ease;
}

.task-enter-from,
.task-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}

.task-move {
  transition: transform 0.5s ease;
}

/* Responsive Design */
@media (max-width: 768px) {
  .todo-container {
    padding: 1.5rem;
  }

  .title {
    font-size: 2rem;
  }

  .stats-section {
    grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  }
}

@media (max-width: 480px) {
  .todo-container {
    padding: 1rem;
  }

  .input-container {
    flex-direction: column;
  }

  .add-button {
    width: 100%;
    justify-content: center;
  }




}

</style>