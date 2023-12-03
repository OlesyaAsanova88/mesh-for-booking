<script setup>
import { ref, computed, watch } from 'vue';

const zones = ['VIP', 'Standard', 'Camp'];
const selectedZone = ref(''); // Добавляем реактивную переменную для выбранной зоны
const bookingTime = ref(''); // Время бронирования
const showBookingTimeAlert = ref(false); // Показывать ли предупреждение

const showBookingInfo = ref(false); // Показывать информацию о бронировании
const bookedSeats = ref([]); // Список забронированных мест

const timeOptions = [
  { value: '1h', text: '1 час' },
  { value: '2h', text: '2 часа' },
  { value: '3h', text: '3 часа' }
];

const seats = ref([
  // Генерируем данные мест для примера
  ...Array.from({ length: 60 }, (_, i) => ({
    label: `Seat ${i + 1}`,
    selected: false,
    booked: false,
    zone: zones[i % zones.length]
  }))
]);

// Функция для фильтрации мест по зоне
function seatsByZone(zone) {
  return seats.value.filter(seat => seat.zone === zone);
}

function seatClasses(seat) {
  return {
    seat: true,
    selected: seat.selected,
    booked: seat.booked,
    disabled: seat.booked,
    vip: seat.zone === 'VIP',
    standard: seat.zone === 'Standard',
    camp: seat.zone === 'Camp'
  };
}

function selectSeat(seat) {
  if (!seat.booked) {
    seat.selected = !seat.selected;
    selectedZone.value = seat.zone; // Обновляем выбранную зону при выборе места
  }
}

function bookSeats() {
  if (!bookingTime.value) {
    showBookingTimeAlert.value = true;
    return;
  }

  bookedSeats.value = seats.value.filter(seat => seat.selected).map(seat => seat.label);
  seats.value.forEach(seat => {
    if (seat.selected) {
      seat.booked = true;
      seat.selected = false;
    }
  });
  showBookingInfo.value = true;
 
}

// Сброс предупреждения при выборе времени
watch(bookingTime, (newVal) => {
  if (newVal) {
    showBookingTimeAlert.value = false;
  }
});

const selectedSeatsInfo = computed(() => {
  return seats.value.filter(s => s.selected).map(s => s.label);
});

</script>

<template>
  <main>
    <div class="zone-selector">
     
      <div class="zone-selector-block">
        <div>Выбранная зона: <strong>{{ selectedZone }}</strong></div>
        <div>Выбранные места: <span>{{ selectedSeatsInfo.join(', ') }}</span></div>
      </div>
      <div class="booking-info" v-if="showBookingInfo">
        <p>Бронирование успешно!</p>
        <p>Время: {{ bookingTime }}</p>
        <p>Места: {{ bookedSeats.join(', ') }}</p>
      </div>
    </div>

    <div class="booking-form">
      <div class="time-selection">
        <label class="custom-radio-button" v-for="option in timeOptions" :key="option.value">
          <input type="radio" v-model="bookingTime" :value="option.value">
          <span class="radio-button"></span>
          {{ option.text }}
        </label>
      </div>
      <button @click="bookSeats" :disabled="!bookingTime" class="booking-form-btn">Забронировать выбранные места</button>
      <div v-if="showBookingTimeAlert" class="alert">
        Выберите время бронирования
      </div>
    </div>

    <div class="zones">
      <div v-for="zone in zones" :key="zone" class="zone">
        <h2>{{ zone }} Zone</h2>
        <div class="grid">
          <div v-for="(seat, index) in seatsByZone(zone)" :key="index"
               :class="seatClasses(seat)"
               @click="seat.booked ? null : selectSeat(seat)">
            {{ seat.label }}
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>


</style>
