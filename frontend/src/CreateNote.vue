<script setup>

import {ref} from 'vue';
const emit = defineEmits();
const newNoteStatus = ref('unimportant');
const newNoteTitle = ref('');
// Emitting the new note when the "Create new note" button is clicked
const createNote = () => {
  const newNote = {
    title: newNoteTitle.value,
    status: newNoteStatus.value,
  };
  
  // Emit to the parent component (NotesApp) to handle creating the note
  emit('create-note', newNote);

  // Reset the form after creating the note
  newNoteTitle.value = '';
  newNoteStatus.value = 'unimportant';
}

</script>

<template>
    <div class="note unimportant">
        
        <input class="new-note-title" v-model="newNoteTitle" placeholder="Enter note title..."   @keyup.enter="createNote">
        
        <div class="status-select">
            <div>
                <input type="radio" id="unimportant" v-model="newNoteStatus" value="unimportant" />
                <label for="unimportant">Unimportant</label>
            </div>

            <div>
                <input type="radio" id="serious" v-model="newNoteStatus" value="serious" />
                <label for="serious">Serious</label>
            </div>

            <div>
                <input type="radio" id="urgent" v-model="newNoteStatus" value="urgent" />
                <label for="urgent">Urgent</label>
            </div>
        </div>

        <button class="create-btn" @click="createNote">Create new note</button>
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

    & > input {
        padding: 15px 5px 15px 5px;
        border: 0;
        font-size: 20px;
        width: 100%;
        border-radius: 5px;

        &::placeholder {
            font-style: italic;
            color: white;
        }
    }

    & > .create-btn {
        padding: 10px 5px 10px 5px;
        width: 100%;
        background-color: white;
        //color: darken($dark-green, 20%);
        color: $dark-green;
        font-weight: bold;
        font-size: 15px;
    }

    &.unimportant {
      background-color: $dark-green;
      & > input {
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

    .status-select {
        display: flex;
        flex-wrap: wrap;
        gap: 10px;

        margin-top: 10px;
        margin-bottom: 10px;
        
        &  input {
            margin-right: 5px;
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
