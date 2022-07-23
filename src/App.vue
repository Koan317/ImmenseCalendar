<template>
  <div>
    <Calendar :year="testDate.year" :month="testDate.month" :lang="testDate.lang" :selectedDays="testDate.selectedDays"
      :buttonWidth="testDate.buttonWidth" :disabled="testDate.disabled" @onChange="onChange"
      @disabledDays="disabledDays"></Calendar>
  </div>
</template>

<script>
import Calendar from './components/calendar.vue';
export default {
  components: {
    Calendar
  },
  data() {
    return {
      testDate: {
        year: 2019,
        month: 1,
        lang: 'EN',
        selectedDays: [1, 5, 28, 21, 3, 15],
        buttonWidth: 150,
        disable: false
      }
    };
  },
  methods: {
    onChange(event) {
      console.log(event)
    },
    disabledDays(date, callback) {
      callback(date.getTime() > Date.now());
    },
  }
};
</script>

<style>
</style>
