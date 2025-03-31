<script setup>
import {computed} from "vue";

const props = defineProps({
  toDo: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['finishToDo', 'deleteToDo', 'editToDo', 'togglePriority'])

const dateTime = computed(() => {
  const date = new Date(props.toDo.createdAt);
  return date.toLocaleString();
})

function finishToDo() {
  emit('finishToDo', props.toDo)
}

function togglePriority() {
  emit('togglePriority', props.toDo)
}

function editToDo() {
  emit('editToDo', props.toDo)
}

function deleteToDo() {
  emit('deleteToDo', props.toDo)
}
</script>

<template>
  <div class="list-group-item todo-item" :class="{'text-muted': toDo.finished }">
    <div class="todo-item__priority">
      <button class="btn btn-link" @click="togglePriority">
          <span v-if="toDo.priority">
            <i class="fas fa-star fa-fw"></i>
          </span>
        <span v-else>
            <i class="far fa-star fa-fw"></i>
          </span>
      </button>
    </div>

    <div class="todo-item__finished">
      <div class="form-check form-switch">
        <input type="checkbox" class="form-check-input" role="switch" :id="`todo_${toDo.createdAt}_finished`"
               :checked="toDo.finished" @change="finishToDo">
        <label :for="`todo_${toDo.createdAt}_finished`" class="form-check-label"></label>
      </div>
    </div>

    <div class="todo-item__text">
      <p class="mb-0" v-html="toDo.text"></p>
      <small>Created At: {{ dateTime }}</small>
    </div>

    <div class="todo-item__actions">
      <div class="btn-group-vertical">
        <button class="btn btn-secondary" @click="editToDo">
          <i class="fas fa-pen fa-fw"></i>
        </button>
        <button class="btn btn-danger" @click="deleteToDo">
          <i class="fas fa-trash fa-fw"></i>
        </button>
      </div>
    </div>
  </div>
</template>