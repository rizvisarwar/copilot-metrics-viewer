<template>
  <div>
    <div class="tiles-container">
      <v-card elevation="4" color="white" variant="elevated" class="mx-auto my-3" style="width: 300px; height: 175px;">
        <v-card-item>
          <div class="tiles-text">
            <div class="spacing-25" />
            <v-tooltip location="bottom start" open-on-hover open-delay="200" close-delay="200">
              <template #activator="{ props }">
                <div v-bind="props" class="text-h6 mb-1">Avg Active Users per Week</div>
              </template>
              <v-card class="pa-2" style="background-color: #f0f0f0; max-width: 350px;">
                <span class="text-caption" style="font-size: 10px !important;">Average number of active users per week. Calculated by dividing the total number of active users across the period by the number of complete weeks.</span>
              </v-card>
            </v-tooltip>
            <div class="text-caption">
              {{ dateRangeDescription }}
            </div>
            <p class="text-h4">{{ avgUsersPerWeek.toFixed(0) }}</p>
          </div>
        </v-card-item>
      </v-card>

      <v-card elevation="4" color="white" variant="elevated" class="mx-auto my-3" style="width: 300px; height: 175px;">
        <v-card-item>
          <div class="tiles-text">
            <div class="spacing-10" />
            <v-tooltip location="bottom start" open-on-hover open-delay="200" close-delay="200">
              <template #activator="{ props }">
                <div v-bind="props" class="text-h6 mb-1">Total Active Users</div>
              </template>
              <v-card class="pa-2" style="background-color: #f0f0f0; max-width: 350px;">
                <span class="text-caption" style="font-size: 10px !important;">Total number of active users during the specified time period.</span>
              </v-card>
            </v-tooltip>
            <div class="text-caption">
              {{ dateRangeDescription }}
            </div>
            <p class="text-h4">{{ totalActiveUsers }}</p>
          </div>
        </v-card-item>
      </v-card>

      <v-card elevation="4" color="white" variant="elevated" class="mx-auto my-3" style="width: 300px; height: 175px;">
        <v-card-item>
          <div class="tiles-text">
            <div class="spacing-25" />
            <v-tooltip location="bottom start" open-on-hover open-delay="200" close-delay="200">
              <template #activator="{ props }">
                <div v-bind="props" class="text-h6 mb-1">Completed Weeks</div>
              </template>
              <v-card class="pa-2" style="background-color: #f0f0f0; max-width: 350px;">
                <span class="text-caption" style="font-size: 10px !important;">Number of complete weeks (7-day cycles) in the specified time period.</span>
              </v-card>
            </v-tooltip>
            <div class="text-caption">
              {{ dateRangeDescription }}
            </div>
            <p class="text-h4">{{ completedWeeks }}</p>
          </div>
        </v-card-item>
      </v-card>
    </div>
  </div>
</template>

<script lang="ts">
import type { Metrics } from '@/model/Metrics';

export default defineNuxtComponent({
  name: 'AvgUsersPerWeekViewer',
  props: {
    metrics: {
      type: Array as () => Metrics[],
      required: true,
    },
    dateRangeDescription: {
      type: String,
      required: true,
    },
  },
  computed: {
    totalActiveUsers(): number {
      return this.metrics.reduce((total, metric) => total + metric.total_active_users, 0);
    },
    completedWeeks(): number {
      if (this.metrics.length === 0) return 0;
      // Calculate the number of complete weeks (7-day cycles)
      const dayCount = this.metrics.length;
      return Math.floor(dayCount / 7);
    },
    avgUsersPerWeek(): number {
      if (this.completedWeeks === 0) return 0;
      return this.totalActiveUsers / this.completedWeeks;
    },
  },
});
</script>

<style scoped>
.tiles-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;
  padding: 20px;
}

.tiles-text {
  text-align: center;
}

.spacing-25 {
  height: 25px;
}

.spacing-10 {
  height: 10px;
}
</style>
