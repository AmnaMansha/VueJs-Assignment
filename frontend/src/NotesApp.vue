<script setup>
import { ref, onMounted,computed } from 'vue'
import Note from './Note.vue'
import {HOST} from './config.js'
import CreateNote from './CreateNote.vue';
import NoteStatusFilter from './NoteStatusFilter.vue'

const notesList = ref([]);
const currentFilter = ref('all');
// Fetch all notes and their tasks
onMounted(async () => {
  try {
    // First fetch all notes
    const notesResponse = await fetch(`${HOST}/notes`);
    if (!notesResponse.ok) throw new Error('Failed to fetch notes');
    const notes = await notesResponse.json();

    // Then fetch tasks for each note and update the notesList
    const notesWithTasks = await Promise.all(notes.map(async (note) => {
      try {
        const tasksResponse = await fetch(`${HOST}/notes/${note.id}/tasks`);
        if (!tasksResponse.ok) throw new Error(`Failed to fetch tasks for note ${note.id}`);
        const tasks = await tasksResponse.json();
        return { ...note, tasks };
      } catch (error) {
        console.error(`Error fetching tasks for note ${note.id}:`, error);
        return { ...note, tasks: [] }; // Return note with empty tasks array if fetch fails
      }
    }));

    notesList.value = notesWithTasks;
  } catch (error) {
    console.error('Error fetching notes:', error);
    alert('Failed to load notes. Please try again.');
  }
});
const filteredNotes = computed(() => {
  if (currentFilter.value === 'all') {
    return notesList.value;
  }
  return notesList.value.filter(note => note.status === currentFilter.value);
});

const handleFilterChange = (status) => {
  currentFilter.value = status;
};
// Function to handle newly created note
const handleNewNote = async (newNote) => {
  try {
    const response = await fetch(`${HOST}/notes`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(newNote),
    });

    if (!response.ok) throw new Error('Failed to create note');

    // Fetch updated list of notes with their tasks
    const notesResponse = await fetch(`${HOST}/notes`);
    if (!notesResponse.ok) throw new Error('Failed to fetch updated notes');
    const notes = await notesResponse.json();

    // Fetch tasks for each note
    const notesWithTasks = await Promise.all(notes.map(async (note) => {
      const tasksResponse = await fetch(`${HOST}/notes/${note.id}/tasks`);
      const tasks = await tasksResponse.json();
      return { ...note, tasks };
    }));

    notesList.value = notesWithTasks;
  } catch (error) {
    console.error('Error:', error);
    alert('Failed to create note. Please try again.');
  }
};

const updateTasks = (noteId, updatedTasks) => {
  const noteIndex = notesList.value.findIndex((n) => n.id === noteId);
  if (noteIndex !== -1) {
    // Create a new array reference to trigger reactivity
    notesList.value = notesList.value.map((note, index) => {
      if (index === noteIndex) {
        return { ...note, tasks: updatedTasks };
      }
      return note;
    });
  }
};
const deleteNote = async (noteId) => {
  try {
    const response = await fetch(`${HOST}/notes/${noteId}`, {
      method: 'DELETE',
      headers: {
        'Content-Type': 'application/json',
      },
    });

    if (!response.ok) {
      throw new Error('Failed to delete note');
    }

    // Remove the note from the local state
    notesList.value = notesList.value.filter(note => note.id !== noteId);
  } catch (error) {
    console.error('Error deleting note:', error);
    alert('Failed to delete note. Please try again.');
  }
};
</script>

<template>
  <div class="app-container">
    
   <NoteStatusFilter @filter-change="handleFilterChange" />
  <div class="notes-container">
    <Note 
    v-for="note in filteredNotes"
      :key="note.id"
      :note="note"
      @update-tasks="updateTasks(note.id, $event)"
        @delete-note="deleteNote"
     
    ></Note>
    <CreateNote @create-note="handleNewNote"></CreateNote>
    
  </div>
  </div>
</template>


<style lang="scss" scoped>
@use 'assets/mediaQueryScreens.scss' as *;

$gutter-size: 15px;

.notes-container {
  display: flex;
  flex-wrap: wrap;
  gap: $gutter-size;
  
  margin-right: auto;
  margin-left: auto;

  border: 1px solid gray;
  border-radius: 10px;

  padding: $gutter-size;
}

@include smallScreen {
  .notes-container {
    width: 90vw;
  }

}

@include mediumScreen {
  .notes-container {
    width: 80vw;
  }

}

@include largeScreen {
  .notes-container {
    width: 1024px;
  }
}
.note {
  transition: all 0.3s ease;
}

.note-enter-from,
.note-leave-to {
  opacity: 0;
  transform: scale(0.9);
}
</style>
