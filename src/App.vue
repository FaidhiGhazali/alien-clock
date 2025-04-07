<template>
  <div class="container">
    <div class="card">
      <h1>Alien Clock</h1>
      <div class="clock-display">
        <p>
          Alien Date: {{ alienTime.day }}-{{ alienTime.month }}-{{ alienTime.year }}
          {{ alienTime.hour }}:{{ alienTime.minute }}:{{ alienTime.second }}
        </p>
        <p>Earth Time: {{ earthTime }}</p>
      </div>

      <div class="alien-time-inputs">
        <label>Set Alien Date:</label>
        <div class="input-group">
          <input type="number" v-model.number="customDate.year" placeholder="Year" />
          <input type="number" v-model.number="customDate.month" placeholder="Month (1-18)" />
          <input type="number" v-model.number="customDate.day" placeholder="Day" />
        </div>
        <label>Set Alien Time:</label>
        <div class="input-group">
          <input type="number" v-model.number="customTime.hour" placeholder="Hour (0-35)" />
          <input type="number" v-model.number="customTime.minute" placeholder="Minute (0-89)" />
          <input type="number" v-model.number="customTime.second" placeholder="Second (0-89)" />
        </div>
        <button @click="setAlienTime">Set Alien Time</button>
      </div>

      <div class="alarm-input">
        <label>Set Alarm (Alien Time HH:mm):</label>
        <input v-model="alarmInput" @input="formatAlarmInput" maxlength="5" />
        <button @click="setAlarm">Set Alarm</button>
      </div>

      <p v-if="alarmTriggered">🚨 Alarm triggered!</p>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue';

const baseAlienTime = {
  year: 2804,
  month: 18,
  day: 31,
  hour: 2,
  minute: 2,
  second: 88,
};
const baseEarth = new Date(Date.UTC(1970, 0, 1, 0, 0, 0));
const alienToEarthSecond = 0.5;
const SECONDS_PER_MINUTE = 90;
const SECONDS_PER_HOUR = 90 * SECONDS_PER_MINUTE;
const SECONDS_PER_DAY = 36 * SECONDS_PER_HOUR;
const months = [44, 42, 48, 40, 48, 44, 40, 44, 42, 40, 40, 42, 44, 48, 42, 40, 44, 38];
const totalDaysInYear = months.reduce((sum, days) => sum + days, 0);
const SECONDS_PER_YEAR = totalDaysInYear * SECONDS_PER_DAY;

const alienTime = reactive({ ...baseAlienTime });
const earthTime = ref('');
const customDate = reactive({ year: '', month: '', day: '' });
const customTime = reactive({ hour: '', minute: '', second: '' });
const alarmInput = ref('');
const alarmTriggered = ref(false);

const computeAlienDateSeconds = (alien) => {
  let s = 0;
  for (let i = 0; i < alien.month - 1; i++) {
    s += months[i] * SECONDS_PER_DAY;
  }
  s += (alien.day - 1) * SECONDS_PER_DAY;
  s += alien.hour * SECONDS_PER_HOUR;
  s += alien.minute * SECONDS_PER_MINUTE;
  s += alien.second;
  return s;
};

const formatDate = (date) => {
  const day = String(date.getUTCDate()).padStart(2, "0");
  const month = String(date.getUTCMonth() + 1).padStart(2, "0");
  const year = date.getUTCFullYear();
  const hours = String(date.getUTCHours()).padStart(2, "0");
  const minutes = String(date.getUTCMinutes()).padStart(2, "0");
  const seconds = String(date.getUTCSeconds()).padStart(2, "0");
  return `${day}-${month}-${year} ${hours}:${minutes}:${seconds}`;
};

const updateEarthTimeFromAlien = () => {
  const currentAlienSeconds = computeAlienDateSeconds(alienTime);
  const baseAlienSeconds = computeAlienDateSeconds(baseAlienTime);
  const yearDiff = alienTime.year - baseAlienTime.year;
  const fullYearSeconds = yearDiff * SECONDS_PER_YEAR;
  const totalAlienSecondsDiff = fullYearSeconds + (currentAlienSeconds - baseAlienSeconds);
  const totalEarthSeconds = totalAlienSecondsDiff * alienToEarthSecond;

  const earthDate = new Date(baseEarth.getTime());
  earthDate.setSeconds(earthDate.getSeconds() + totalEarthSeconds);
  earthTime.value = formatDate(earthDate);
};

const updateAlienClock = () => {
  alienTime.second++;
  if (alienTime.second >= SECONDS_PER_MINUTE) {
    alienTime.second = 0;
    alienTime.minute++;
  }
  if (alienTime.minute >= 90) {
    alienTime.minute = 0;
    alienTime.hour++;
  }
  if (alienTime.hour >= 36) {
    alienTime.hour = 0;
    alienTime.day++;
  }

  const daysInMonth = months[alienTime.month - 1];
  if (alienTime.day > daysInMonth) {
    alienTime.day = 1;
    alienTime.month++;
  }
  if (alienTime.month > 18) {
    alienTime.month = 1;
    alienTime.year++;
  }

  updateEarthTimeFromAlien();
};

const setAlienTime = () => {
  const { year, month, day } = customDate;
  const { hour, minute, second } = customTime;
  const daysInMonth = months[month - 1];

  if (
    year && month && day &&
    hour !== '' && minute !== '' && second !== '' &&
    month >= 1 && month <= 18 &&
    day >= 1 && day <= daysInMonth &&
    hour >= 0 && hour < 36 &&
    minute >= 0 && minute < 90 &&
    second >= 0 && second < 90
  ) {
    alienTime.year = parseInt(year);
    alienTime.month = parseInt(month);
    alienTime.day = parseInt(day);
    alienTime.hour = parseInt(hour);
    alienTime.minute = parseInt(minute);
    alienTime.second = parseInt(second);
    updateEarthTimeFromAlien();
  } else {
    alert('Invalid Alien Date or Time!');
  }
};

const setAlarm = () => {
  alarmTriggered.value = false;
  if (!alarmInput.value) {
    alert("Please enter an alarm time");
    return;
  }
  const [hour, minute] = alarmInput.value.split(":").map(Number);
  if (hour >= 0 && hour < 36 && minute >= 0 && minute < 90) {
    const interval = setInterval(() => {
      if (alienTime.hour === hour && alienTime.minute === minute) {
        alarmTriggered.value = true;
        clearInterval(interval);
      }
    }, 500);
  } else {
    alert("Invalid Alien Time for Alarm!");
  }
};

const formatAlarmInput = () => {
  let digits = alarmInput.value.replace(/\D/g, '').slice(0, 4);
  if (digits.length >= 3) {
    alarmInput.value = `${digits.slice(0, 2)}:${digits.slice(2)}`;
  } else {
    alarmInput.value = digits;
  }
};

onMounted(() => {
  setInterval(updateAlienClock, 500);
});
</script>


<style>
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
}

.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-image: url('@/assets/cartoon-space-background-q0svwt1fa3jh8t00.jpg');
  background-size: cover;
  background-position: center;
}

.card {
  background-color: rgba(255, 255, 255, 0.95);
  border: 1px solid #ddd;
  border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  padding: 20px 40px;
  max-width: 500px;
  width: 100%;
}

.card * {
  text-align: center;
}

h1 {
  font-size: 2.5em;
  margin-bottom: 20px;
}

.clock-display {
  margin: 20px 0;
  font-size: 1.2em;
}

.alien-time-inputs, .alarm-input {
  margin: 20px 0;
}

.input-group {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  justify-content: center;
}

input {
  padding: 8px;
  font-size: 1em;
  border: 1px solid #ccc;
  border-radius: 5px;
  width: 100px;
}

button {
  margin: 10px;
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
button:hover {
  background-color: #0056b3;
}

p {
  font-size: 1.2em;
}

.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}

.dialog-box {
  background-color: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  max-width: 300px;
  text-align: center;
}
</style>
