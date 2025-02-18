<template>
  <div class="calendar-container">
    <FullCalendar
      :options="calendarOptions"
      class="calendar"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import FullCalendar from '@fullcalendar/vue3';
import dayGridPlugin from '@fullcalendar/daygrid';
import interactionPlugin from '@fullcalendar/interaction';

const emit = defineEmits(['date-selected']);
const selectedDate = ref(null);

const calendarOptions = {
  plugins: [dayGridPlugin, interactionPlugin],
  initialView: 'dayGridMonth',
  locale: 'zh-cn',
  dateClick: (info) => {
    emit('date-selected', info.dateStr);
    selectedDate.value = info.dateStr;
  },
  headerToolbar: {
    left: 'prev,next today',
    center: 'title',
    right: 'dayGridMonth'
  },
  height: 'auto',
  dayMaxEvents: true,
  weekNumbers: false,
  fixedWeekCount: false,
  showNonCurrentDates: false,
  buttonText: {
    today: '今天'
  },
  dayCellDidMount: (arg) => {
    const today = new Date();
    if (arg.date.toDateString() === today.toDateString()) {
      arg.el.classList.add('today-cell');
    }
  }
};
</script>

<style scoped>
.calendar-container {
  flex: 0 0 auto;
  max-width: 400px;
  height: auto;
  background: #333;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(255, 255, 255, 0.1);
  padding: 10px;
  margin-left: auto;
}

.calendar {
  font-family: Arial, sans-serif;
  color: #fff;
}

:deep(.fc) {
  height: 100%;
  color: #fff;
}

:deep(.fc-header-toolbar) {
  margin-bottom: 1.5em !important;
  padding: 0.5em;
  color: #fff;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

:deep(.fc-header-toolbar .fc-title) {
  white-space: nowrap;
}

:deep(.fc-button) {
  background-color: #4CAF50 !important;
  border-color: #4CAF50 !important;
  text-transform: capitalize !important;
  padding: 8px 16px !important;
  font-weight: normal !important;
  color: #fff;
}

:deep(.fc-button:hover) {
  background-color: #45a049 !important;
  border-color: #45a049 !important;
}

:deep(.fc-button-active) {
  background-color: #388E3C !important;
  border-color: #388E3C !important;
}

:deep(.fc-day-today) {
  background-color: #444 !important;
}

:deep(.fc-day-header) {
  padding: 10px 0 !important;
  font-weight: bold !important;
  color: #fff !important;
}

:deep(.fc-day) {
  cursor: pointer;
  transition: background-color 0.2s;
  color: #fff;
}

:deep(.fc-day:hover) {
  background-color: #555;
}

:deep(.fc-daygrid-day-number) {
  padding: 8px !important;
  color: #fff;
}

:deep(.today-cell) {
  background-color: #444 !important;
  font-weight: bold;
}

:deep(.fc-day-other) {
  background-color: #222;
}
</style>