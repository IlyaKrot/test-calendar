<template>
  <div class="calendar">
    <div class="calendar__locale">
      <div
          :class="['calendar__locale-btn', { 'calendar__locale-btn--active': locale === 'ru' }]"
          @click="changeLocale('ru')"
      >
        RU
      </div>
      <div
          :class="['calendar__locale-btn', { 'calendar__locale-btn--active': locale === 'en' }]"
          @click="changeLocale('en')"
      >
        EN
      </div>
    </div>

    <div class="calendar__wrapper">
      <div class="calendar__header">
        <div class="calendar__btn" @click="changeMonth(-1)">
          ❰
        </div>
        <div class="calendar__month">
          {{ locales[locale].months[currentMonth] }} {{ currentYear }}
        </div>
        <div class="calendar__btn" @click="changeMonth(1)">
          ❱
        </div>
      </div>

      <div class="calendar__body">
        <div class="calendar__weekdays">
          <div class="calendar__weekday" v-for="(day, index) in locales[locale].days" :key="index">
            {{ day }}
          </div>
        </div>

        <div class="calendar__dates">
          <div
              v-for="d in calendarDays"
              :key="d.date.toISOString()"
              @click="selectDate(d)"
              class="calendar__date"
              :class="[
                {'calendar__date--disabled': !d.current },
                {'calendar__date--selected':
                  selectedYear === d.date.getFullYear() &&
                  selectedMonth === d.date.getMonth() &&
                  selectedDay === d.date.getDate()
                },
                {'calendar__date--today':
                  d.date.getFullYear() === new Date().getFullYear() &&
                  d.date.getMonth() === new Date().getMonth() &&
                  d.date.getDate() === new Date().getDate()
                }
              ]"
          >
            {{ d.day }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import {computed, ref, watch} from "vue";
import {locales} from "@/models/calendar";

type LocaleType = 'en' | 'ru';
type dateType = {
  day: number;
  current: boolean;
  date: Date;
};

const props = defineProps<{
  date: string
}>()

const emit = defineEmits<{
  (e: 'changeDate', day: Date): void
}>()

const locale = ref<LocaleType>('en');

const selectedDate = ref<Date>(props.date ? new Date(props.date) : new Date());
const selectedYear = ref<number>(selectedDate.value.getFullYear());
const selectedMonth = ref<number>(selectedDate.value.getMonth());
const selectedDay = ref<number>(selectedDate.value.getDate());

const currentYear = ref<number>(selectedYear.value);
const currentMonth = ref<number>(selectedMonth.value);
// const currentDay = ref<number>(selectedDay.value);

watch(() => props.date, (newDate: string): void => {
  selectedDate.value = newDate ? new Date(newDate) : new Date();
  selectedYear.value = selectedDate.value.getFullYear();
  selectedMonth.value = selectedDate.value.getMonth();
  selectedDay.value = selectedDate.value.getDate();
});

const calendarDays = computed(() => {
  const year = currentYear.value;
  const month = currentMonth.value;

  const daysInCurrent = new Date(year, month + 1, 0).getDate();
  const firstWeekday = new Date(year, month, 1).getDay();
  const daysInPrev = new Date(year, month, 0).getDate();

  const days: dateType[] = [];

  for (let i = firstWeekday - 1; i >= 0; i--) {
    const d = daysInPrev - i;
    days.push({ day: d, current: false, date: new Date(year, month - 1, d) });
  }

  for (let d = 1; d <= daysInCurrent; d++) {
    days.push({ day: d, current: true, date: new Date(year, month, d) });
  }

  const total = days.length;
  const trailing = (7 - (total % 7)) % 7;
  for (let d = 1; d <= trailing; d++) {
    days.push({ day: d, current: false, date: new Date(year, month + 1, d) });
  }

  return days;
});

const changeLocale = (loc: LocaleType) => {
  locale.value = loc;
};

const changeMonth = (delta: number): void => {
  let newMonth = currentMonth.value + delta;
  let newYear = currentYear.value;

  if (newMonth < 0) {
    newMonth = 11;
    newYear -= 1;
  } else if (newMonth > 11) {
    newMonth = 0;
    newYear += 1;
  }

  currentMonth.value = newMonth;
  currentYear.value = newYear;
};

const selectDate = (d: dateType): void => {
  emit('changeDate', d.date);
};

</script>

<style scoped lang="scss">
.calendar {
  width: 300px;
  border: 1px solid #ccc;
  border-radius: 8px;
  overflow: hidden;
  font-family: Arial, sans-serif;

  &__btn {
    cursor: pointer;
    user-select: none;
    padding: 0 8px;
    font-size: 1.2em;

    &:hover {
      color: #007bff;
    }
  }

  &__locale {
    display: flex;
    justify-content: flex-end;
    padding: 8px;
    background-color: #f5f5f5;
  }

  &__locale-btn {
    cursor: pointer;
    margin-left: 8px;
    padding: 4px 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    background-color: #fff;
    user-select: none;

    &--active {
      background-color: #e6f0ff;
      border-color: #7aa7ff;
    }
  }

  &__wrapper {
    padding: 16px;
  }

  &__header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 16px;
  }

  &__month {
    font-weight: bold;
    font-size: 1.2em;
  }

  &__year {
    font-size: 1.2em;
  }

  &__body {
    display: flex;
    flex-direction: column;
  }

  &__weekdays {
    display: flex;
    justify-content: space-between;
    margin-bottom: 8px;
  }

  &__weekday {
    width: calc(100% / 7);
    text-align: center;
    font-weight: 600;
  }

  &__dates {
    display: flex;
    flex-wrap: wrap;
  }

  &__date {
    width: calc(100% / 7);
    text-align: center;
    padding: 6px 0;

    cursor: pointer;

    &:hover {
      background-color: #f0f0f0;
    }

    &--selected {
      background-color: #007bff;
      color: #fff;
      border-radius: 50%;
    }

    &--today {
      border: 1px solid #007bff;
      border-radius: 50%;
    }

    &--disabled {
      color: #ccc;
      pointer-events: none;
    }
  }
}
</style>