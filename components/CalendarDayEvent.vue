<template>
  <CalendarDay
    @toggleExpandedWeek="$emit('toggleExpandedWeek')"
    :showExpandedWeek="showExpandedWeek"
    :day="day"
  >
    <div class="day-blocks">
      <TimeLines
        v-if="showTimelines"
        slot="timelines"
        class="sim-calendar--grid--day--timelines"
        :showHalfHourTicks="false"
        @createTimeBlock="createPendingEvent"
      />
      <div class="local--day--blocks local--day--event-blocks">
        <template v-for="(event, index) in events">
          <TimeBlockEvent
            :block="event"
            ref="timeBlockEvent"
            @updatePosition="updateBlockPosition('event', index)"
            @click.native.stop="selectEvent(event, index)"
          />
        </template>
      </div>
      <div class="local--day--blocks local--day--pending-blocks">
        <TimeBlockPendingEvent
          v-if="pendingEvent"
          ref="pendingEvent"
          :block="pendingEvent"
          @click.native.stop="selectPendingEvent(pendingEvent)"
          @updatePosition="updateBlockPosition('pendingEvent')"
          @updateTimeBlock="updatePendingEvent"
          @clearPendingEvent="$emit('clearPendingEvent')"
        />
      </div>
      <div
        class="local--day--blocks local--day--aggregate-blocks"
        @click="expandWeek"
       >
        <template v-if="blocks">
          <div v-for="(block, index) in blocks">
            <!--
            <TimeBlockSpecificAvailability
              v-if="hasOnlySpecificInstructors(block)"
              :key="index"
              :block="block"
              @click.native="createPendingEvent(block.startTime)"
            />
            -->
            <TimeBlockAggregateAvailability
              :key="index"
              :block="block"
              @click.native="createPendingEvent(block.startTime)"
            />
          </div>
        </template>
        <template v-else><TimeBlockNull /></template>
      </div>
    </div>
  </CalendarDay>
</template>

<script>
  import CalendarDay from './CalendarDay'
  import TimeLines from './TimeLines'
  import TimeBlockAggregateAvailability from './TimeBlockAggregateAvailability'
  import TimeBlockSpecificAvailability from './TimeBlockSpecificAvailability'
  import TimeBlockPendingEvent from './TimeBlockPendingEvent'
  import TimeBlockEvent from './TimeBlockEvent'
  import TimeBlockNull from './TimeBlockNull'

  import { map } from 'lodash/fp'

  export default {
    components: {
      CalendarDay,
      TimeLines,
      TimeBlockAggregateAvailability,
      TimeBlockSpecificAvailability,
      TimeBlockPendingEvent,
      TimeBlockNull,
      TimeBlockEvent,
    },
    extends: CalendarDay,
    props: {
      day: Object,
      availabilities: Array,
      showExpandedWeek: Boolean,
      pendingEvent: Object,
      events: Array,
    },
    computed: {
      dateService() {
        return this.$store.state.services.date
      },
      isSelected() {
        return this.dateService.selectedDate.isSame(this.day, 'day')
      },
      showTimelines() {
        return this.isSelected && this.showExpandedWeek
      },
      blocks() {
        return map(availability => {
          return {
            startTime: availability,
            duration: 0.5,
          }
        })(this.availabilities)
      },
    },
    methods: {
      createPendingEvent(startTime) {
        this.$emit('createPendingEvent', this.day, startTime)
      },
      updateBlockPosition(type, index) {
        const element = type === "pendingEvent"
          ? this.$refs.pendingEvent
          : this.$refs.timeBlockEvent[index]
        this.$emit('updateBlockPosition', {
          domPosition: element.$el.getBoundingClientRect(),
          offset: {
            x: 0,
            y: 0,
          },
        })
      },
      updatePendingEvent(block) {
        this.$emit('updatePendingEvent', block)
      },
      hasOnlySpecificInstructors(block) {
          return block.specificInstructors.length
          && !block.generalInstructors.length
      },
      selectPendingEvent(event) {
        this.$emit('selectPendingEvent', event)
        this.updateBlockPosition('pendingEvent')
      },
      selectEvent(event, index) {
        this.$emit('selectEvent', event)
        this.updateBlockPosition('event', index)
      },
    },
  }
</script>

<style lang="scss">
  @import '../styles/calendar-day';
  .local--day--event-blocks, .local--day--pending-blocks {
    position: absolute;
    width: 100%;
    height: 100%;
    margin: 0 !important;
  }
  .sim-calendar-day {
    .day-blocks {
      width: 100%;
      margin-left: 20%;
      position: relative;
    }
  }
</style>
