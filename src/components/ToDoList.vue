<script setup>
import ToDoListItem from "./ToDoListItem.vue";
import {inject} from "vue";

defineProps({
  list: {
    type: Array,
    required: true,
  },
  listKeyPrefix: {
    type: String,
    required: true
  }
})
const animationKey = inject('animationKey')

const emit = defineEmits(['finishToDo', 'togglePriority', 'editToDo', 'deleteToDo']);

function finishToDo(toDo) {
  emit('finishToDo', toDo)
}

function togglePriority(toDo) {
  emit('togglePriority', toDo);
}

function editToDo(toDo) {
  emit('editToDo', toDo);
}

function deleteToDo(toDo) {
  emit('deleteToDo', toDo)
}
</script>

<template>
  <div class="list-group">
    <slot name="listHeader"></slot>

    <TransitionGroup :name="animationKey">
      <template v-for="toDo in list" :key="`${listKeyPrefix}_${toDo.createdAt}`">
        <ToDoListItem :toDo="toDo" @finishToDo="finishToDo" @togglePriority="togglePriority"
                      @editToDo="editToDo" @deleteToDo="deleteToDo"/>
      </template>

      <div v-if="$slots.emptyListItem && list.length === 0" class="list-group-item-success">
        <slot name="emptyListItem"></slot>
      </div>
    </TransitionGroup>
  </div>
</template>