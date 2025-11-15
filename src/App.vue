<template>
    <Calendar :date="date" @change-date="changeDate"></Calendar>
</template>

<script setup lang="ts">
import Calendar from "@/components/Calendar.vue";
import {ref} from "vue";

const defaultDate = "2025-11-11";

const date = ref<string>(defaultDate);

const changeDate = (d: Date): void => {
  const pad = (n: number) => String(n).padStart(2, "0");
  date.value = `${d.getFullYear()}-${pad(d.getMonth() + 1)}-${pad(d.getDate())}`;
  console.log(date.value);
}
</script>

<style scoped>
</style>
