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
                <span class="text-caption" style="font-size: 10px !important;">Average number of unique active users per week. Calculated from the sum of weekly unique users across completed weeks.</span>
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
import type { Seat } from '@/model/Seat';
import { parseUtcDate } from '@/utils/dateUtils';

export default defineNuxtComponent({
  name: 'AvgUsersPerWeekViewer',
  props: {
    metrics: {
      type: Array as () => Metrics[],
      required: true,
    },
    seats: {
      type: Array as () => Seat[],
      required: false,
      default: () => [],
    },
    dateRangeDescription: {
      type: String,
      required: true,
    },
  },
  computed: {
    totalUniqueActiveUsers(): number {
      if (this.seats.length > 0) return this.seats.length;
      return this.weeklyUniqueActiveUsersSum;
    },
    weeklyUniqueActiveUsersSum(): number {
      if (this.completedWeeks === 0) return 0;
      const { startDate } = this.getDateRange();
      const metricsByDate = this.metrics.map(metric => ({
        date: parseUtcDate(metric.day),
        totalActiveUsers: metric.total_active_users,
      }));
      let total = 0;
      for (let weekIndex = 0; weekIndex < this.completedWeeks; weekIndex += 1) {
        const weekStart = new Date(startDate.getTime() + weekIndex * 7 * 24 * 60 * 60 * 1000);
        const weekEnd = new Date(weekStart.getTime() + 6 * 24 * 60 * 60 * 1000);
        const weekMax = metricsByDate
          .filter(entry => entry.date >= weekStart && entry.date <= weekEnd)
          .reduce((max, entry) => Math.max(max, entry.totalActiveUsers), 0);
        total += weekMax;
      }
      return total;
    },
    completedWeeks(): number {
      if (this.metrics.length === 0) return 0;
      const { startDate, endDate } = this.getDateRange();
      const dayCount = this.getInclusiveDayCount(startDate, endDate);
      return Math.floor(dayCount / 7);
    },
    avgUsersPerWeek(): number {
      if (this.completedWeeks === 0) return 0;
      return this.weeklyUniqueActiveUsersSum / this.completedWeeks;
    },
  },
  methods: {
    getDateRange(): { startDate: Date; endDate: Date } {
      const dates = this.metrics.map(metric => parseUtcDate(metric.day).getTime());
      const startDate = new Date(Math.min(...dates));
      const endDate = new Date(Math.max(...dates));
      return { startDate, endDate };
    },
    getInclusiveDayCount(startDate: Date, endDate: Date): number {
      const startUtc = Date.UTC(startDate.getUTCFullYear(), startDate.getUTCMonth(), startDate.getUTCDate());
      const endUtc = Date.UTC(endDate.getUTCFullYear(), endDate.getUTCMonth(), endDate.getUTCDate());
      const diffDays = Math.floor((endUtc - startUtc) / (24 * 60 * 60 * 1000));
      return diffDays + 1;
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
