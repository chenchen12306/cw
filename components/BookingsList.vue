<template>
  <div class="bookings-list">
    <h2>预订列表</h2>
    <div class="list-container">
      <div v-if="bookings.length === 0" class="no-bookings">
        暂无预订记录
      </div>
      <div v-else class="booking-item" v-for="booking in bookings" :key="booking.id">
        <div class="booking-header">
          <span class="room-name">{{ booking.roomName }}</span>
          <span class="booking-date">{{ formatDate(booking.date) }}</span>
        </div>
        <div class="booking-details">
          <p>预订人: {{ booking.customerName }}</p>
          <p>时间: {{ booking.startTime }} - {{ booking.endTime }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'BookingsList',
  data() {
    return {
      bookings: []
    }
  },
  methods: {
    formatDate(date) {
      return new Date(date).toLocaleDateString('zh-CN')
    }
  }
}
</script>

<style scoped>
.bookings-list {
  padding: 1rem;
}

.list-container {
  margin-top: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 1rem;
}

.no-bookings {
  text-align: center;
  color: #666;
  padding: 2rem;
}

.booking-item {
  border-bottom: 1px solid #eee;
  padding: 1rem 0;
}

.booking-item:last-child {
  border-bottom: none;
}

.booking-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.5rem;
}

.room-name {
  font-weight: bold;
  color: #2c3e50;
}

.booking-date {
  color: #666;
}

.booking-details {
  color: #666;
  font-size: 0.9rem;
}

.booking-details p {
  margin: 0.2rem 0;
}
</style> 