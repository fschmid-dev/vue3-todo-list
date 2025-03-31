<script setup>
import {computed, onMounted, provide, ref, toRaw, watch} from "vue";
import Swal from "sweetalert2";
import localforage from "localforage";
import ToDoList from "./components/ToDoList.vue";

const localforage_key = 'vue3-todo-list';
const animationKey = ref('');
provide('animationKey', animationKey)

const toDos = ref([]);

const activeToDos = computed(() => {
  return toDos.value.filter(toDo => toDo.finished !== true).sort(sortToDosByCreatedAt)
})

const finishedToDos = computed(() => {
  return toDos.value.filter(toDo => toDo.finished === true).sort(sortToDosByCreatedAt)
})

function sortToDosByCreatedAt(a, b) {
  if (a.priority && !b.priority) {
    return -1;
  }

  if (!a.priority && b.priority) {
    return 1;
  }

  return a.createdAt < b.createdAt ? 1 : -1;
}

function showNewToDoModal() {
  Swal.fire({
    title: 'Create new To-Do',
    html: 'Enter the new To-Do',
    showConfirmButton: true,
    confirmButtonText: 'Create',
    confirmButtonColor: 'var(--bs-success)',
    confirmButtonAriaLabel: 'Create To-Do',
    showCancelButton: true,
    cancelButtonText: 'Cancel',
    cancelButtonColor: 'var(--bs-danger)',
    cancelButtonAriaLabel: 'Cancel',
    input: 'text',
    inputValue: '',
    inputPlaceholder: 'Enter the To-Do Task',
    inputAutoTrim: true,
    inputAutoFocus: true,
    preConfirm: (text) => {
      if (!text) {
        return Swal.showValidationMessage('To-Do can\'t be empty!')
      }

      createToDoCallback(text)
    }
  })
}

function createToDoCallback(text) {
  const toDo = {
    createdAt: Date.now(),
    text: text,
    finished: false
  }

  toDos.value.push(toDo);
}

function showEditToDoModal(toDoToEdit) {
  const toDoIndex = findToDoIndexByCreatedAt(toDoToEdit.createdAt)
  if (toDoIndex === -1) {
    return;
  }

  Swal.fire({
    title: 'Edit To-Do',
    text: 'Enter the new To-Do',
    showConfirmButton: true,
    confirmButtonText: 'Save',
    confirmButtonColor: 'var(--bs-success)',
    confirmButtonAriaLabel: 'Save To-Do',
    showCancelButton: true,
    cancelButtonText: 'Cancel',
    cancelButtonColor: 'var(--bs-danger)',
    cancelButtonAriaLabel: 'Cancel',
    input: 'text',
    inputValue: toDoToEdit.text,
    inputAutoFocus: true,
    inputAutoTrim: true,
    inputPlaceholder: 'Enter the To-Do',
    preConfirm: (newText) => {
      if (!newText) {
        return Swal.showValidationMessage('To-Do can\'t be empty!')
      }

      if (newText === toDoToEdit.text) {
        // No changes, so nothing to save
        return;
      }

      const toDo = toDos.value[toDoIndex];
      toDo.text = newText
    }
  })
}

function showDeleteToDoConfirm(toDoToDelete) {
  Swal.fire({
    title: 'Delete To-Do?',
    html: `Do you really want to delete the To-Do<br><b>"${toDoToDelete.text}"</b><br>This action can\'t be reverted!`,
    icon: 'warning',
    showConfirmButton: true,
    confirmButtonText: 'Delete',
    confirmButtonAriaLabel: 'Delete To-Do',
    confirmButtonColor: 'var(--bs-primary)',
    showCancelButton: true,
    cancelButtonText: 'Cancel',
    cancelButtonColor: 'var(--bs-danger)',
    cancelButtonAriaLabel: 'Cancel Delete To-Do',
    preConfirm: () => {
      const toDoIndex = findToDoIndexByCreatedAt(toDoToDelete.createdAt)
      if (toDoIndex === -1) {
        return
      }

      toDos.value.splice(toDoIndex, 1);

      Swal.fire({
        title: 'To-Do deleted!',
        icon: 'success',
        timer: 2000,
        timerProgressBar: true,
        confirmButtonText: 'Close',
        confirmButtonColor: 'var(--bs-primary)',
      })
    }
  })
}

function finishToDo(toDoToFinish) {
  const toDoIndex = findToDoIndexByCreatedAt(toDoToFinish.createdAt)
  if (toDoIndex === -1) {
    return;
  }

  const toDo = toDos.value[toDoIndex];
  toDo.finished = !toDo.finished;
}

function toggleToDoPriority(toDoToToggle) {
  const toDoIndex = findToDoIndexByCreatedAt(toDoToToggle.createdAt)
  if (toDoIndex === -1) {
    return;
  }

  const toDo = toDos.value[toDoIndex];
  toDo.priority = !toDo.priority;
}

function findToDoIndexByCreatedAt(createdAt) {
  return toDos.value.findIndex(toDo => toDo.createdAt === createdAt)
}

watch(toDos, () => {
  const toDosToSave = toRaw(toDos.value);

  localforage.setItem(localforage_key, toDosToSave).then(value => {
    console.debug('To-Dos saved!', value);
  }).catch(err => {
    console.error('Error saving To-Dos to storage!', err);
  })
}, {deep: true})

onMounted(() => {
  localforage.getItem(localforage_key).then(savedToDos => {
    if (savedToDos) {
      toDos.value = savedToDos
    }
    console.debug('To-Dos loaded!', savedToDos)
  }).catch(err => {
    console.error('Error loading To-Dos from storage!', err);
  })

  setTimeout(() => {
    animationKey.value = 'fade';
  }, 100)
})
</script>

<template>
  <div class="container my-3">
    <div class="d-flex flex-row justify-content-between align-items-center">
      <h1>Vue3 To-Do-List</h1>
      <button class="btn btn-primary" @click="showNewToDoModal">
        <i class="fas fa-plus fa-fw"></i>
        Create To-Do
      </button>
    </div>
    <hr>
    <ToDoList :list="activeToDos" list-key-prefix="open_todo"
              @finishToDo="finishToDo" @editToDo="showEditToDoModal"
              @deleteToDo="showDeleteToDoConfirm" @togglePriority="toggleToDoPriority">
      <template v-slot:listHeader>
        <div class="list-group-item list-group-item-primary">
          <h2>Open To-Dos</h2>
        </div>
      </template>

      <template v-slot:emptyListItem>
        <div class="list-group-item list-group-item-success">
          All To-Dos completed!
        </div>
      </template>
    </ToDoList>

    <Transition name="fade">
      <div v-if="finishedToDos.length > 0">
        <hr>
        <ToDoList list-key-prefix="finished_todo" :list="finishedToDos"
                  @finishToDo="finishToDo" @editToDo="showEditToDoModal"
                  @deleteToDo="showDeleteToDoConfirm" @togglePriority="toggleToDoPriority">
          <template v-slot:listHeader>
            <div class="list-group-item list-group-item-secondary">
              <h2>Finished To-Dos</h2>
            </div>
          </template>
        </ToDoList>
      </div>
    </Transition>
  </div>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>