<template>
    <li class="task-item" :class="{ 'completed': task.done }">
      <div class="task-content">
        <input
          type="checkbox"
          :checked="task.done"
          @change="toggleTask"
          class="task-checkbox"
          :id="'task-' + index"
        />
        <label :for="'task-' + index" class="task-text">
          {{ task.text }}
          <span v-if="task.createdAt" class="task-date">
            {{ formatDate(task.createdAt) }}
          </span>
        </label>
      </div>
      
      <div class="task-buttons">
        <button 
          @click="toggleTask" 
          :class="['btn', 'btn-toggle', { 'btn-done': task.done }]"
          :title="task.done ? 'Tandai belum selesai' : 'Tandai selesai'"
        >
          <span class="btn-icon">{{ task.done ? '↺' : '✓' }}</span>
          {{ task.done ? "Batal" : "Selesai" }}
        </button>
        
        <button 
          class="btn btn-delete" 
          @click="confirmDelete"
          title="Hapus tugas"
        >
          <span class="btn-icon">🗑</span>
          Hapus
        </button>
      </div>
    </li>
  </template>
  
  <script>
import Swal from 'sweetalert2'; // Tambahkan import SweetAlert2

export default {
  name: 'TaskItem',
  
  props: {
    task: {
      type: Object,
      required: true,
      validator: (task) => {
        return task.hasOwnProperty('text') && task.hasOwnProperty('done');
      }
    },
    index: {
      type: Number,
      required: true
    }
  },

  methods: {
    toggleTask() {
      this.$emit('toggle-task', this.index);
    },

    confirmDelete() {
      // Gunakan SweetAlert2 untuk konfirmasi hapus
      Swal.fire({
        title: 'Konfirmasi Hapus',
        text: 'Apakah Anda yakin ingin menghapus tugas ini?',
        icon: 'warning',
        showCancelButton: true,
        confirmButtonText: 'Ya, hapus!',
        cancelButtonText: 'Batal'
      }).then((result) => {
        if (result.isConfirmed) {
          this.$emit('delete-task', this.index);
          Swal.fire(
            'Dihapus!',
            'Tugas telah dihapus.',
            'success'
          );
        }
      });
    },

    formatDate(date) {
      if (!date) return '';
      const d = new Date(date);
      return d.toLocaleDateString('id-ID', {
        day: 'numeric',
        month: 'short',
        hour: '2-digit',
        minute: '2-digit'
      });
    }
  }
};
</script>


  <style scoped>
  .task-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #3a3f47;
    padding: 12px 16px;
    margin: 8px 0;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    transition: all 0.3s ease;
  }
  
  .task-item:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
  }
  
  .task-content {
    display: flex;
    align-items: center;
    flex: 1;
    margin-right: 16px;
    gap: 12px;
  }
  
  .task-checkbox {
    appearance: none;
    width: 20px;
    height: 20px;
    border: 2px solid #42b983;
    border-radius: 4px;
    cursor: pointer;
    position: relative;
    transition: all 0.2s ease;
  }
  
  .task-checkbox:checked {
    background-color: #42b983;
  }
  
  .task-checkbox:checked::after {
    content: '✓';
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    color: white;
    font-size: 14px;
  }
  
  .task-text {
    color: #ffffff;
    font-size: 16px;
    cursor: pointer;
    display: flex;
    flex-direction: column;
    gap: 4px;
  }
  
  .task-date {
    font-size: 12px;
    color: #888;
  }
  
  .completed .task-text {
    text-decoration: line-through;
    color: #888;
  }
  
  .task-buttons {
    display: flex;
    gap: 8px;
  }
  
  .btn {
    display: inline-flex;
    align-items: center;
    padding: 6px 12px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
    transition: all 0.2s ease;
    background-color: transparent;
    color: white;
  }
  
  .btn-icon {
    margin-right: 4px;
    font-size: 16px;
  }
  
  .btn-toggle {
    background-color: #42b983;
  }
  
  .btn-toggle:hover {
    background-color: #3aa876;
  }
  
  .btn-done {
    background-color: #666;
  }
  
  .btn-done:hover {
    background-color: #555;
  }
  
  .btn-delete {
    background-color: #ff5252;
  }
  
  .btn-delete:hover {
    background-color: #ff3838;
  }
  
  @media (max-width: 480px) {
    .task-item {
      flex-direction: column;
      align-items: stretch;
      gap: 12px;
    }
  
    .task-buttons {
      justify-content: flex-end;
    }
  
    .btn {
      padding: 8px 16px;
    }
  }
  </style>
