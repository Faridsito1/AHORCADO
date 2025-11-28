<template>
  <button class="btn" @click="$emit('click')">
    <q-icon :name="icon" size="24px" v-if="icon" />
    <span>{{ label }}</span>
  </button>
</template>

<script setup>
defineProps({
  label: String,
  icon: String
});
</script>

<style scoped>
.btn {
  background: #232323;
  color: white;
  padding: 12px 22px;
  border-radius: 10px;
  border: none;
  font-size: 18px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  transition: transform .2s;
}

.btn:hover {
  transform: scale(1.05);
}
</style>

