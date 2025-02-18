<template>
  <div class="app">
    <div class="main-layout">
      <div class="content-section">
        <RoomStatus :selected-date="selectedDate" />
      </div>
      <div class="calendar-section">
        <MainCalendar @date-selected="onDateSelected" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import MainCalendar from '../components/MainCalendar.vue'
import RoomStatus from '../components/RoomStatus.vue'

const selectedDate = ref("");

const onDateSelected = (date) => {
  selectedDate.value = date;
};
</script>

<style scoped>
.app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #2c3e50;
  margin: 20px;
}

.main-layout {
  display: flex;
  gap: 20px;
  max-width: 1600px;
  margin: 0 auto;
}

.content-section {
  flex: 1;
  padding: 20px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.calendar-section {
  flex: 0 0 auto;
}
</style>
