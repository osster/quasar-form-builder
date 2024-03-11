<template>
  <div class="form-builder-dateTime"
       :class="customClass">
    <div class="outsideLabel">{{ placeholder ? label : null }}</div>
    <div :key="rendrKey"
         class="dateTime-input">
      <q-input v-show="show('date')"
               v-model="dateTime.date"
               :name="name"
               class="form-calender"
               :clearable="clearable"
               :loading="loading"
               :filled="filled"
               readonly
               mask="date"
               dir="ltr"
               :disable="disable"
               :label="placeholder ? null : label"
               :stack-label="!!placeholder"
               :placeholder="placeholder"
               :rules="rules"
               :lazy-rules="lazyRules"
               :outlined="outlined"
               :class="customClass"
               :input-class="customClass"
               @click="showDateMenu">
        <template #prepend>
          <q-icon :name="calendarIcon"
                  class="cursor-pointer"
                  :class="customClass">
            <q-menu v-if="!readonly"
                    v-model="showDate"
                    :class="customClass">
              <q-date v-model="dateTime.date"
                      :calendar="calendar"
                      mask="YYYY/MM/DD"
                      :range="range"
                      :multiple="multiple"
                      :disable="disable"
                      :title="title ? title : label"
                      :today-btn="todayBtn"
                      :class="customClass"
                      @update:model-value="change($event)">
                <div class="row items-center justify-end">
                  <q-btn v-close-popup
                         label="Close"
                         color="primary"
                         flat />
                </div>
              </q-date>
            </q-menu>
          </q-icon>
        </template>
      </q-input>
      <q-input v-show="show('time')"
               v-model="dateTime.time"
               :name="name"
               class="time-input-dateTime"
               :clearable="clearable"
               dir="ltr"
               :disable="disable"
               :filled="filled"
               :stack-label="!!placeholder"
               :label="show('date') || placeholder ? null:label"
               :placeholder="!show('date') ? placeholder:null"
               mask="time"
               :rules="rules"
               :lazy-rules="lazyRules"
               readonly
               :outlined="outlined"
               :class="customClass"
               :input-class="customClass"
               @click="showTimeMenu">
        <template #append>
          <q-menu v-if="!readonly"
                  v-model="showTime"
                  :class="customClass">
            <q-time v-model="dateTime.time"
                    mask="HH:mm:00"
                    format24h
                    :disable="disable"
                    :title="title ? title : label"
                    :now-btn="nowBtn"
                    :class="customClass">
              <div class="row items-center justify-end">
                <q-btn v-close-popup
                       label="Close"
                       color="primary"
                       flat />
              </div>
            </q-time>
          </q-menu>
          <q-icon :name="clockIcon"
                  class="cursor-pointer" />
        </template>
      </q-input>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import inputMixin from '../mixins/inputMixin.js'
export default {
  name: 'FormBuilderDateTime',
  mixins: [inputMixin],
  props: {
    name: {
      default: '',
      type: String
    },
    calendar: {
      default: 'en',
      type: String
    },
    calendarIcon: {
      default: 'event',
      type: String
    },
    clockIcon: {
      default: 'access_time',
      type: String
    },
    title: {
      default: '',
      type: String
    },
    placeholder: {
      default: '',
      type: String
    },
    value: {
      default: '',
      type: String
    },
    nowBtn: {
      default: false,
      type: Boolean
    },
    todayBtn: {
      default: false,
      type: Boolean
    }
  },
  data() {
    return {
      dateTime: {
        date: '',
        time: ''
      },
      rendrKey: Date.now(),
      showDate: false,
      showTime: false
    }
  },
  watch: {
    inputData(newValue) {
      this.updateDateTime(newValue)
    },
    dateTime: {
      handler(newValue) {
        const newDate = newValue.date
        const newTime = newValue.time
        if (this.type === 'dateTime' && this.isValidDate(newDate) && this.isValidTime(newTime)) {
          this.inputData = newDate + ' ' + newTime
          this.change(this.inputData)
        } else if (this.type === 'date' && this.isValidDate(newDate)) {
          this.inputData = newDate
          this.change(this.inputData)
        } else if (this.type === 'time' && this.isValidTime(newTime)) {
          this.inputData = newTime
          this.change(this.inputData)
        }
      },
      deep: true
    },
    value(newValue) {
      this.updateDateTime(newValue)
    }
  },
  methods: {
    getNormalizedTimeString (dateObject) {
      const newDateObject = new Date(dateObject)
      return newDateObject.getHours().toString().padStart(2, '0') + ':' + newDateObject.getMinutes().toString().padStart(2, '0') + ':' + newDateObject.getSeconds().toString().padStart(2, '0')
    },
    getNormalizedDateString (dateObject) {
      const newDateObject = new Date(dateObject)
      return newDateObject.getFullYear().toString().padStart(4, '0') + '-' + (newDateObject.getMonth() + 1).toString().padStart(2, '0') + '-' + newDateObject.getDate().toString().padStart(2, '0')
    },
    updateDateTime(newDate) {
      this.dateTime = {
        date: '',
        time: ''
      }

      if (!newDate) {
        return
      }

      const newDateObject = new Date(newDate)

      if (this.type === 'dateTime') {
        const dateData = this.getNormalizedDateString(newDateObject)
        const time = this.getNormalizedTimeString(newDateObject)
        if (!this.isValidDate(dateData) || !this.isValidTime(time)) {
          return
        }
        this.dateTime.date = dateData
        this.dateTime.time = time
      } else if (this.type === 'time') {
        if (!this.isValidTime(newDate)) {
          return
        }
        this.dateTime.time = newDate
      } else if (this.type === 'date') {
        if (!this.isValidDate(newDate)) {
          return
        }
        this.dateTime.date = newDate
      }

      this.rendrKey = Date.now()
    },
    isValidDate(dateDate) {
      return (/[1,2][0,9][0-9][0-9]-[0,1]{0,1}[0-9]-[0-3]{0,1}[0-9]/gm).test(dateDate)
    },
    isValidTime(time) {
      return (/[0,1,2]{0,1}[0-9]:[0-5]{0,1}[0-9]:[0-5]{0,1}[0-9]/gm).test(time)
    },
    showDateMenu() {
      this.showDate = true
    },
    showTimeMenu() {
      this.showTime = true
    },
    show(t) {
      return this.type === 'dateTime' || this.type === t
    },
    formatTime(time) {
      return moment(time, 'HH:mm').format('HH:mm:00')
    },
    getTime(time) {
      return moment(time, 'HH:mm').format('HH:mm:00')
    }
  }
}
</script>

<style scoped lang="scss">
.dateTime-input {
  display: flex;
  flex-direction: row;

  label {
    width: 100%;
  }

  .time-input-dateTime {
    .q-field__native {
      padding: 24px 0 8px;
    }
  }
}

// removing dotted border for readonly fields from project
:deep(.q-field--outlined.q-field--readonly .q-field__control:before) {
  border-style: solid;
}

:deep(.q-field--standard.q-field--readonly .q-field__control:before) {
  border-bottom: 1px solid rgba(0, 0, 0, 0.24);
  transition: border-color 0.36s cubic-bezier(0.4, 0, 0.2, 1);
}
</style>
