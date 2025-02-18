<template>
  <div v-if="selectedDate" class="room-status">
    <!-- 移动统计报表到顶部 -->
    <div class="status-report">
      <div class="report-header">
        <h3>床号状态统计</h3>
        <button class="clear-btn" @click="clearAllData">清除所有记录</button>
      </div>
      <div class="report-grid">
        <div class="report-item available">
          <span class="count">{{ statusCounts.available }}</span>
          <span class="label">可预订</span>
        </div>
        <div class="report-item booked">
          <span class="count">{{ statusCounts.booked }}</span>
          <span class="label">已预订</span>
        </div>
        <div class="report-item occupied">
          <span class="count">{{ statusCounts.occupied }}</span>
          <span class="label">已入住</span>
        </div>
      </div>
    </div>

    <h3 class="date-header">{{ selectedDate }} 状态</h3>
    <div class="rooms-grid">
      <div v-for="room in rooms" :key="room.id" class="room-card">
        <h3>{{ room.number }}</h3>
        <div class="time-slots">
          <!-- 上午时段 -->
          <div class="time-slot">
            <div class="time-label">上午</div>
            <div 
              class="status" 
              :class="room.morning.status"
              @click="showRoomDetail(room, 'morning')"
            >
              {{ getStatusText(room.morning.status) }}
            </div>
            <div v-if="room.morning.booking" class="booking-info">
              <p>预订人: {{ room.morning.booking.guestName }}</p>
            </div>
            <div class="room-actions">
              <button 
                v-if="room.morning.status === 'available'" 
                class="action-btn book-btn"
                @click="openBookingDialog(room, 'morning')"
              >
                预订
              </button>
              <button 
                v-if="room.morning.status === 'booked'" 
                class="action-btn occupy-btn"
                @click="openCheckInDialog(room, 'morning')"
              >
                入住
              </button>
              <button 
                v-if="room.morning.status === 'occupied'" 
                class="action-btn checkout-btn"
                @click="openCheckOutDialog(room, 'morning')"
              >
                退床
              </button>
            </div>
          </div>
          
          <!-- 下午时段 -->
          <div class="time-slot">
            <div class="time-label">下午</div>
            <div 
              class="status" 
              :class="room.afternoon.status"
              @click="showRoomDetail(room, 'afternoon')"
            >
              {{ getStatusText(room.afternoon.status) }}
            </div>
            <div v-if="room.afternoon.booking" class="booking-info">
              <p>预订人: {{ room.afternoon.booking.guestName }}</p>
            </div>
            <div class="room-actions">
              <button 
                v-if="room.afternoon.status === 'available'" 
                class="action-btn book-btn"
                @click="openBookingDialog(room, 'afternoon')"
              >
                预订
              </button>
              <button 
                v-if="room.afternoon.status === 'booked'" 
                class="action-btn occupy-btn"
                @click="openCheckInDialog(room, 'afternoon')"
              >
                入住
              </button>
              <button 
                v-if="room.afternoon.status === 'occupied'" 
                class="action-btn checkout-btn"
                @click="openCheckOutDialog(room, 'afternoon')"
              >
                退床
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 修改预订对话框 -->
    <div v-if="showBooking" class="dialog-overlay">
      <div class="dialog">
        <h3>预订 {{ selectedRoom?.number }}</h3>
        <form @submit.prevent="handleBooking">
          <!-- 添加日期时间选择 -->
          <div class="form-group date-range">
            <div class="date-time-group">
              <label>入住时间:</label>
              <div class="date-time-inputs">
                <input 
                  type="date" 
                  v-model="bookingForm.checkIn"
                  :min="props.selectedDate"
                  required
                >
                <select v-model="bookingForm.checkInTime" required>
                  <option value="morning">上午</option>
                  <option value="afternoon">下午</option>
                </select>
              </div>
            </div>
            <div class="date-time-group">
              <label>退房时间:</label>
              <div class="date-time-inputs">
                <input 
                  type="date" 
                  v-model="bookingForm.checkOut"
                  :min="bookingForm.checkIn"
                  required
                >
                <select v-model="bookingForm.checkOutTime" required>
                  <option value="morning">上午</option>
                  <option value="afternoon">下午</option>
                </select>
              </div>
            </div>
          </div>
          <!-- 其他表单字段保持不变 -->
          <div class="form-group">
            <label>预订人姓名:</label>
            <input v-model="bookingForm.guestName" required>
          </div>
          <div class="form-group">
            <label>主治医生:</label>
            <input v-model="bookingForm.doctorName" required>
          </div>
          <div class="form-group">
            <label>诊断:</label>
            <textarea v-model="bookingForm.diagnosis" rows="3" required></textarea>
          </div>
          <div class="form-group">
            <label>其他备注:</label>
            <textarea v-model="bookingForm.notes" rows="3"></textarea>
          </div>
          <div class="dialog-buttons">
            <button type="button" class="cancel-btn" @click="closeDialog">取消</button>
            <button type="submit" class="confirm-btn">确认预订</button>
          </div>
        </form>
      </div>
    </div>

    <!-- 修改房间详情对话框 -->
    <div v-if="showDetail" class="dialog-overlay">
      <div class="dialog">
        <h3>详情</h3>
        <div class="room-detail">
          <p><strong>编号:</strong> {{ selectedRoom?.number }}</p>
          <p><strong>时段:</strong> {{ selectedTimeSlot === 'morning' ? '上午' : '下午' }}</p>
          <p><strong>状态:</strong> {{ getStatusText(selectedRoom && selectedRoom[selectedTimeSlot]?.status) }}</p>
          <div v-if="selectedRoom && selectedRoom[selectedTimeSlot]?.booking">
            <h4>预订信息</h4>
            <p><strong>预订人:</strong> {{ selectedRoom[selectedTimeSlot].booking.guestName }}</p>
            <p><strong>主治医生:</strong> {{ selectedRoom[selectedTimeSlot].booking.doctorName }}</p>
            <p><strong>诊断:</strong> {{ selectedRoom[selectedTimeSlot].booking.diagnosis }}</p>
            <p><strong>入住时间:</strong> {{ selectedRoom[selectedTimeSlot].booking.checkIn }}
              {{ selectedRoom[selectedTimeSlot].booking.checkInTime === 'morning' ? '上午' : '下午' }}</p>
            <p><strong>退房时间:</strong> {{ selectedRoom[selectedTimeSlot].booking.checkOut }}
              {{ selectedRoom[selectedTimeSlot].booking.checkOutTime === 'morning' ? '上午' : '下午' }}</p>
            <p v-if="selectedRoom[selectedTimeSlot].booking.notes">
              <strong>备注:</strong> {{ selectedRoom[selectedTimeSlot].booking.notes }}
            </p>
            
            <!-- 添加管理员操作按钮和密码验证 -->
            <div v-if="selectedRoom[selectedTimeSlot].status === 'booked'" class="admin-actions">
              <div v-if="!showPasswordInput" class="action-buttons">
                <button class="cancel-btn" @click="showPasswordInput = true">
                  撤销预订
                </button>
              </div>
              <div v-else class="password-verification">
                <input 
                  type="password" 
                  v-model="adminPassword"
                  placeholder="请输入管理员密码"
                  class="password-input"
                >
                <div class="verification-buttons">
                  <button class="confirm-btn" @click="verifyAndCancel">确认</button>
                  <button class="cancel-btn" @click="cancelPasswordInput">取消</button>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="dialog-buttons">
          <button class="confirm-btn" @click="closeDialog">关闭</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watchEffect } from 'vue';

const props = defineProps({
  selectedDate: {
    type: String,
    required: true
  }
});

// 修改基础房间数据结构
const createEmptyTimeSlot = () => ({
  status: 'available',
  booking: null
});

const baseRooms = [
  { id: 1, number: '+12床' },
  { id: 2, number: '+15床' },
  { id: 3, number: '+16床' },
  { id: 4, number: '+17床' },
  { id: 5, number: '+18床' },
  { id: 6, number: '+19床' },
  { id: 7, number: '+20床' },
  { id: 8, number: '+21床' },
  { id: 9, number: '+22床' },
  { id: 10, number: '11' },
  { id: 11, number: '12' },
  { id: 12, number: '33' },
  { id: 13, number: '35' }
].map(room => ({
  ...room,
  morning: createEmptyTimeSlot(),
  afternoon: createEmptyTimeSlot()
}));

// 修改 bookingHistory 的初始化，从 localStorage 加载数据
const bookingHistory = ref(new Map(
  JSON.parse(localStorage.getItem('bookingHistory') || '[]')
));

// 添加保存到 localStorage 的函数
const saveToLocalStorage = () => {
  localStorage.setItem(
    'bookingHistory', 
    JSON.stringify(Array.from(bookingHistory.value))
  );
};

// 当前显示的房间数据
const rooms = ref([]);

// 添加状态统计的计算
const statusCounts = ref({
  available: 0,
  booked: 0,
  occupied: 0
});

// 添加选中的时段
const selectedTimeSlot = ref('morning');

// 初始化房间状态
const initializeRooms = () => {
  const currentDate = props.selectedDate;
  if (!bookingHistory.value.has(currentDate)) {
    // 如果该日期没有预订记录，创建新的记录
    const newRooms = baseRooms.map(room => ({
      ...room,
      morning: createEmptyTimeSlot(),
      afternoon: createEmptyTimeSlot()
    }));
    bookingHistory.value.set(currentDate, newRooms);
    saveToLocalStorage(); // 保存新的状态
  }
  // 更新当前显示的房间数据
  rooms.value = bookingHistory.value.get(currentDate);
};

// 更新状态统计
const updateStatusCounts = () => {
  const counts = {
    available: 0,
    booked: 0,
    occupied: 0
  };
  
  rooms.value.forEach(room => {
    counts[room.morning.status]++;
    counts[room.afternoon.status]++;
  });
  
  statusCounts.value = counts;
};

// 修改 watchEffect，添加状态统计更新
watchEffect(() => {
  if (props.selectedDate) {
    if (!bookingHistory.value.has(props.selectedDate)) {
      initializeRooms();
    } else {
      rooms.value = bookingHistory.value.get(props.selectedDate);
    }
    updateStatusCounts(); // 更新状态统计
  }
});

// 对话框状态
const showBooking = ref(false);
const showDetail = ref(false);
const selectedRoom = ref(null);

// 添加管理员状态（实际项目中应该从用户系统获取）
const isAdmin = ref(true); // 临时设置为 true 用于测试

// 修改预订表单数据结构
const bookingForm = ref({
  guestName: '',
  doctorName: '',
  diagnosis: '',
  notes: '',
  checkIn: '',
  checkOut: '',
  checkInTime: 'morning',
  checkOutTime: 'afternoon'
});

// 添加密码验证相关的状态
const showPasswordInput = ref(false);
const adminPassword = ref('');
const ADMIN_PASSWORD = '123456'; // 实际项目中应该从后端获取或使用更安全的方式

// 状态文本映射
const getStatusText = (status) => {
  const statusMap = {
    available: '可预订',
    booked: '已预订',
    occupied: '已占用'
  };
  return statusMap[status];
};

// 修改打开预订对话框函数
const openBookingDialog = (room, timeSlot) => {
  selectedRoom.value = room;
  selectedTimeSlot.value = timeSlot;
  showBooking.value = true;
  bookingForm.value = {
    guestName: '',
    doctorName: '',
    diagnosis: '',
    notes: '',
    checkIn: '',
    checkOut: '',
    checkInTime: 'morning',
    checkOutTime: 'afternoon'
  };
};

// 打开房间详情
const showRoomDetail = (room, timeSlot) => {
  selectedRoom.value = room;
  selectedTimeSlot.value = timeSlot;
  showDetail.value = true;
};

// 验证密码并撤销预订
const verifyAndCancel = () => {
  if (adminPassword.value === ADMIN_PASSWORD) {
    cancelBooking(selectedRoom.value, selectedTimeSlot.value);
    showPasswordInput.value = false;
    adminPassword.value = '';
  } else {
    alert('密码错误！');
  }
};

// 取消密码输入
const cancelPasswordInput = () => {
  showPasswordInput.value = false;
  adminPassword.value = '';
};

// 修改撤销预订函数
const cancelBooking = (room, timeSlot) => {
  if (!room || !timeSlot) return;
  
  if (confirm(`确定要撤销 ${room[timeSlot].booking.guestName} 的预订吗？`)) {
    const updatedRoom = {
      ...room,
      [timeSlot]: {
        status: 'available',
        booking: null
      }
    };
    
    const roomIndex = rooms.value.findIndex(r => r.id === room.id);
    if (roomIndex !== -1) {
      const updatedRooms = [...rooms.value];
      updatedRooms[roomIndex] = updatedRoom;
      rooms.value = updatedRooms;
      bookingHistory.value.set(props.selectedDate, updatedRooms);
      saveToLocalStorage();
      updateStatusCounts();
      closeDialog();
    }
  }
};

// 修改预订处理函数
const handleBooking = () => {
  if (!validateBookingDates()) return;
  
  const checkIn = new Date(bookingForm.value.checkIn);
  const checkOut = new Date(bookingForm.value.checkOut);
  
  // 遍历预订日期范围
  for (let date = new Date(checkIn); date <= checkOut; date.setDate(date.getDate() + 1)) {
    const currentDate = formatDate(date);
    const dateRooms = bookingHistory.value.get(currentDate) || [...baseRooms];
    const roomIndex = dateRooms.findIndex(r => r.id === selectedRoom.value.id);
    
    if (roomIndex !== -1) {
      const updatedRooms = [...dateRooms];
      const room = updatedRooms[roomIndex];
      
      // 判断是否为入住首日或退房末日，相应更新上/下午时段
      const isCheckInDay = currentDate === bookingForm.value.checkIn;
      const isCheckOutDay = currentDate === bookingForm.value.checkOut;
      
      if (isCheckInDay) {
        if (bookingForm.value.checkInTime === 'morning') {
          room.morning = {
            status: 'booked',
            booking: { ...bookingForm.value }
          };
        }
        if (bookingForm.value.checkInTime === 'afternoon' || 
            (isCheckInDay === isCheckOutDay && bookingForm.value.checkOutTime === 'afternoon')) {
          room.afternoon = {
            status: 'booked',
            booking: { ...bookingForm.value }
          };
        }
      } else if (isCheckOutDay) {
        if (bookingForm.value.checkOutTime === 'afternoon') {
          room.afternoon = {
            status: 'booked',
            booking: { ...bookingForm.value }
          };
        }
        if (bookingForm.value.checkOutTime === 'morning') {
          room.morning = {
            status: 'booked',
            booking: { ...bookingForm.value }
          };
        }
      } else {
        // 中间日期全天预订
        room.morning = {
          status: 'booked',
          booking: { ...bookingForm.value }
        };
        room.afternoon = {
          status: 'booked',
          booking: { ...bookingForm.value }
        };
      }
      
      bookingHistory.value.set(currentDate, updatedRooms);
      if (currentDate === props.selectedDate) {
        rooms.value = updatedRooms;
      }
    }
  }
  
  saveToLocalStorage();
  updateStatusCounts();
  closeDialog();
};

// 修改预订验证函数
const validateBookingDates = () => {
  const checkIn = new Date(bookingForm.value.checkIn);
  const checkOut = new Date(bookingForm.value.checkOut);
  
  if (isNaN(checkIn.getTime()) || isNaN(checkOut.getTime())) {
    alert('请选择有效的日期');
    return false;
  }

  if (checkOut < checkIn) {
    alert('退房日期不能早于入住日期');
    return false;
  }

  // 检查日期范围内的房间可用性
  for (let date = new Date(checkIn); date <= checkOut; date.setDate(date.getDate() + 1)) {
    const currentDate = formatDate(date);
    const dateRooms = bookingHistory.value.get(currentDate);
    if (dateRooms) {
      const room = dateRooms.find(r => r.id === selectedRoom.value.id);
      if (room) {
        const isCheckInDay = currentDate === bookingForm.value.checkIn;
        const isCheckOutDay = currentDate === bookingForm.value.checkOut;
        
        // 检查具体时段是否可用
        if (isCheckInDay) {
          if (bookingForm.value.checkInTime === 'morning' && room.morning.status !== 'available') {
            alert(`${currentDate} 上午该房间已被预订`);
            return false;
          }
          if (bookingForm.value.checkInTime === 'afternoon' && room.afternoon.status !== 'available') {
            alert(`${currentDate} 下午该房间已被预订`);
            return false;
          }
        } else if (isCheckOutDay) {
          if (bookingForm.value.checkOutTime === 'morning' && room.morning.status !== 'available') {
            alert(`${currentDate} 上午该房间已被预订`);
            return false;
          }
          if (bookingForm.value.checkOutTime === 'afternoon' && room.afternoon.status !== 'available') {
            alert(`${currentDate} 下午该房间已被预订`);
            return false;
          }
        } else {
          // 中间日期需要全天可用
          if (room.morning.status !== 'available' || room.afternoon.status !== 'available') {
            alert(`${currentDate} 该房间已被预订`);
            return false;
          }
        }
      }
    }
  }
  
  return true;
};

// 关闭所有对话框
const closeDialog = () => {
  showBooking.value = false;
  showDetail.value = false;
  selectedRoom.value = null;
  showPasswordInput.value = false;
  adminPassword.value = '';
  bookingForm.value = {
    guestName: '',
    doctorName: '',
    diagnosis: '',
    notes: '',
    checkIn: '',
    checkOut: '',
    checkInTime: 'morning',
    checkOutTime: 'afternoon'
  };
};

// 入住处理
const openCheckInDialog = (room, timeSlot) => {
  if (confirm(`确认办理入住？\n编号: ${room.number}\n预订人: ${room[timeSlot].booking?.guestName}`)) {
    const currentDate = props.selectedDate;
    const dateRooms = bookingHistory.value.get(currentDate);
    const roomIndex = dateRooms.findIndex(r => r.id === room.id);

    if (roomIndex !== -1) {
      const updatedRooms = [...dateRooms];
      updatedRooms[roomIndex] = {
        ...updatedRooms[roomIndex],
        [timeSlot]: {
          ...updatedRooms[roomIndex][timeSlot],
          status: 'occupied'
        }
      };

      bookingHistory.value.set(currentDate, updatedRooms);
      rooms.value = updatedRooms;
      saveToLocalStorage();
      updateStatusCounts();
    }
  }
};

// 退床处理
const openCheckOutDialog = (room, timeSlot) => {
  if (confirm(`确认办理退床？\n编号: ${room.number}`)) {
    const currentDate = props.selectedDate;
    const dateRooms = bookingHistory.value.get(currentDate);
    const roomIndex = dateRooms.findIndex(r => r.id === room.id);

    if (roomIndex !== -1) {
      const updatedRooms = [...dateRooms];
      updatedRooms[roomIndex] = {
        ...updatedRooms[roomIndex],
        [timeSlot]: {
          ...updatedRooms[roomIndex][timeSlot],
          status: 'available',
          booking: null
        }
      };

      bookingHistory.value.set(currentDate, updatedRooms);
      rooms.value = updatedRooms;
      saveToLocalStorage();
      updateStatusCounts();
    }
  }
};

// 添加日期工具函数
const formatDate = (date) => {
  const d = new Date(date);
  const year = d.getFullYear();
  const month = String(d.getMonth() + 1).padStart(2, '0');
  const day = String(d.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
};

// 检查日期范围内是否有冲突
const checkDateRangeAvailability = (roomId, startDate, endDate) => {
  const start = new Date(startDate);
  const end = new Date(endDate);
  
  for (let date = new Date(start); date <= end; date.setDate(date.getDate() + 1)) {
    const currentDate = formatDate(date);
    if (bookingHistory.value.has(currentDate)) {
      const dateRooms = bookingHistory.value.get(currentDate);
      const room = dateRooms.find(r => r.id === roomId);
      if (room && room.status !== 'available') {
        return false;
      }
    }
  }
  return true;
};

// 添加清除数据的功能（可选）
const clearAllData = () => {
  if (confirm('确定要清除所有预订记录吗？')) {
    bookingHistory.value = new Map();
    localStorage.removeItem('bookingHistory');
    initializeRooms();
    updateStatusCounts();
  }
};
</script>

<style scoped>
.room-status {
  display: flex;
  flex-direction: column;
  gap: 20px;
  background-color: #000;
  color: #fff;
}

.date-header {
  margin: 0;
  padding: 0 20px;
}

.status-report {
  background: #333;
  color: #fff;
  border-radius: 8px;
  padding: 20px;
}

.report-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.report-header h3 {
  margin: 0;
}

.report-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
}

.rooms-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr); /* 一行显示7个房间 */
  gap: 10px;
  padding: 20px;
}

.room-card {
  background: #444; /* 更深的背景色 */
  color: #fff;
  border: 1px solid #666; /* 较浅的边框 */
  border-radius: 8px;
  padding: 10px;
  box-shadow: 0 2px 4px rgba(255, 255, 255, 0.1);
  display: flex;
  flex-direction: column;
  gap: 10px;
  transition: transform 0.2s; /* 添加过渡效果 */
}

.room-card:hover {
  transform: scale(1.05); /* 鼠标悬停时放大 */
}

.room-card h3 {
  margin: 0;
  font-size: 1em;
  color: #fff;
}

.time-label {
  font-size: 0.9em;
  color: #fff;
  margin-bottom: 5px;
}

.status {
  padding: 6px;
  border-radius: 4px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
}

.status:hover {
  opacity: 0.8;
}

.available {
  background-color: #4caf50;
  color: white;
}

.booked {
  background-color: #ff9800;
  color: white;
}

.occupied {
  background-color: #f44336;
  color: white;
}

.room-actions {
  margin-top: 5px;
  display: flex;
  justify-content: center;
  gap: 5px;
}

.action-btn {
  padding: 4px 8px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.8em;
  transition: all 0.3s ease;
}

.action-btn:hover {
  opacity: 0.8;
}

.book-btn {
  background-color: #4caf50;
  color: white;
}

.occupy-btn {
  background-color: #ff9800;
  color: white;
}

.checkout-btn {
  background-color: #f44336;
  color: white;
}

.clear-btn {
  padding: 8px 16px;
  background-color: #dc3545;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9em;
  transition: all 0.3s ease;
}

.clear-btn:hover {
  opacity: 0.8;
}

.booking-info {
  font-size: 0.9em;
  color: #666;
  border-top: 1px solid #eee;
  padding-top: 8px;
}

.booking-info p {
  margin: 4px 0;
}

.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.dialog {
  background: #333;
  color: #fff;
  padding: 20px;
  border-radius: 8px;
  min-width: 300px;
  max-width: 500px;
  box-shadow: 0 2px 10px rgba(255, 255, 255, 0.1);
}

.form-group {
  margin: 15px 0;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  color: #fff;
}

.form-group input {
  width: 100%;
  padding: 8px;
  border: 1px solid #444;
  border-radius: 4px;
}

.dialog-buttons {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.confirm-btn, .cancel-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.confirm-btn {
  background: #4caf50;
  color: white;
}

.cancel-btn {
  background: #f44336;
  color: white;
}

.room-detail {
  margin: 15px 0;
  line-height: 1.6;
  background: #444;
}

.room-detail h4 {
  margin: 15px 0 10px;
  color: #fff;
}

.date-time {
  margin-bottom: 15px;
}

.date-time-inputs {
  display: flex;
  gap: 10px;
}

.date-time-inputs input[type="date"] {
  flex: 2;
}

.date-time-inputs select {
  flex: 1;
  padding: 8px;
  border: 1px solid #444;
  border-radius: 4px;
}

textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #444;
  border-radius: 4px;
  resize: vertical;
}

.admin-actions {
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid #444;
}

.admin-actions button {
  width: 100%;
}

.time-slots {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.time-slot {
  border: 1px solid #444;
  border-radius: 4px;
  padding: 10px;
}

.password-verification {
  margin-top: 15px;
  padding: 15px;
  border: 1px solid #444;
  border-radius: 4px;
  background: #444;
}

.password-input {
  width: 100%;
  padding: 8px;
  margin-bottom: 10px;
  border: 1px solid #444;
  border-radius: 4px;
}

.verification-buttons {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
}

.action-buttons {
  margin-top: 15px;
}
</style>