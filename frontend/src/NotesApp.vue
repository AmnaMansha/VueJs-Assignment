<script setup>
import { ref, onMounted } from 'vue'
import Note from './Note.vue'
import {HOST} from './config.js'
import CreateNote from './CreateNote.vue';


const notesList = ref([]);


onMounted(async () => {
  notesList.value = await (await fetch(`${HOST}/notes`)).json();
})
// Function to handle newly created note
const handleNewNote = async (newNote) => {
  // Call API to create a new note
  await fetch(`${HOST}/notes`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(newNote),
  });

  // Fetch the updated list of notes
  notesList.value = await (await fetch(`${HOST}/notes`)).json();
}

</script>

<template>
  <div class="notes-container">
    <Note 
      v-for="note in notesList" 
      :key="note.id" 
      :note="note"
    ></Note>
    <CreateNote @create-note="handleNewNote"></CreateNote>
    
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
</style>
