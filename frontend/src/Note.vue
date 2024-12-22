<script setup>
import {HOST} from './config.js';
import {ref} from 'vue';

const props = defineProps({
  note: {
    type: Object,
    required: true
  }
});
const newTaskContent = ref(""); // State to hold the new task input
const emit = defineEmits(['update-tasks','delete-note']);

// Function to create a new task for the note
const createTask = async (noteId) => {
  if (!newTaskContent.value.trim()) {
    alert("Task content cannot be empty.");
    return;
  }

  try {
    const response = await fetch(`${HOST}/notes/${noteId}/tasks`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        content: newTaskContent.value,
      }),
    });

    if (!response.ok) {
      const error = await response.json();
      alert(`Error: ${error.error}`);
      return;
    }

    const createdTask = await response.json();
    // Create a new array with all existing tasks plus the new one
    const updatedTasks = [...props.note.tasks, createdTask];
    emit('update-tasks', updatedTasks);
    newTaskContent.value = ""; // Clear the input field
  } catch (error) {
    console.error("Error creating task:", error);
    alert("Failed to create task. Please try again.");
  }
};

// Function to delete a task from the note

const deleteTask = async (taskId) => {
  try {
    const response = await fetch(`${HOST}/tasks/${taskId}`, {
      method: "DELETE",
    });

    if (!response.ok) {
      alert("Failed to delete task. Please try again.");
      return;
    }

    // Filter out the deleted task and emit the updated tasks
    const updatedTasks =props.note.tasks.filter(task => task.id !== taskId);
    emit('update-tasks', updatedTasks);
  } catch (error) {
    console.error("Error deleting task:", error);
    alert("Failed to delete task. Please try again.");
  }
};
const handleDeleteNote = () => {
  if (confirm('Are you sure you want to delete this note?')) {
    emit('delete-note', props.note.id);
  }
};

</script>

<template>
    <div :class="['note',note.status,note.id]">
        <div class="delete-button" @click="handleDeleteNote">&#x1F5D1;</div>
        <h2>{{note.title}}</h2>

        <div class="tasks">
          <div class="task" v-for="task in note.tasks" :key="task.id">
            <div class="content">{{ task.content }}</div>
            <div class="delete-button"  @click="deleteTask(task.id)">&#x1F5D1;</div>
          </div>

          <div class="new-task">
           
            <input
          v-model="newTaskContent"
          class="content"
          placeholder="Enter new task..."
            @keyup.enter="createTask(note.id)"
        />
            <button class="create-btn" @click="createTask(note.id)">+</button>
          </div>
        </div>
    </div>


</template>


<style lang="scss" scoped>
@use 'assets/mediaQueryScreens.scss' as *;
@use 'assets/colors.scss' as *;

$gutter-size: 15px;


.note {
    min-height: 200px;
    border-radius: 10px;
    padding: 10px;
    color: white;
    position: relative;

    & > .delete-button {
      position: absolute;
      top: 5px;
      right: 10px;
      font-size: 25px;
      cursor: pointer;
    }


    h2 {
      padding-bottom: 5px;
      width: calc(100% - 25px);
    }

    .task {
      padding: 10px 5px 10px 5px;
      margin-bottom: 10px;
    }

  

    .task {
      border-radius: 5px;
      display: flex;
      align-items: center;

      & > .content {
        flex-grow: 1;
      }

      & > .delete-button {
        visibility: hidden;
        flex-grow: 0;
        color: black;
        font-size: 20px;
        cursor: pointer;
      }


      &:hover > .delete-button {
        visibility: visible;
      }
    }

  
    .new-task {
      display: flex;  

      & > input { 
        flex-grow: 1;
        border: 0;
        padding: 15px 5px 15px 5px;

        &::placeholder {
          color: white;
        }
      }

      & > .create-btn {
        background-color: white;
        font-size: 25px;
        font-weight: bold;
        width: 40px;
      }
    }
  

    &.unimportant {
      background-color: $dark-green;
      .task, .new-task > input {
        background-color: $light-green;
        color: darken($dark-green, 20%);
      }

      .new-task > .create-btn {
        color: $dark-green;
      }
    }

    &.serious {
      background-color: $dark-yellow;

      .task, .new-task > input {
        background-color: $light-yellow;
        color: darken($dark-yellow, 20%);
      }

      .new-task > .create-btn {
        color: $dark-yellow;
      }
    }

    &.urgent {
      background-color: $dark-red;

      .task, .new-task > input {
        background-color: $light-red;
        color: darken($dark-red, 20%);      }

      .new-task > .create-btn {
        color: $dark-red;
      }
    }


}

@include smallScreen {
  .note {
    width: 100%;
  }
}

@include mediumScreen {
  .note {
    width: calc((100% - $gutter-size) / 2);
  }
}

@include largeScreen {
  .note {
    width: calc((100% - ($gutter-size * 2)) / 3);
  }
}
</style>
