<script setup lang="ts">
import { ref, onMounted } from 'vue';
import AppLayout from '@/layouts/AppLayout.vue';

import CashFlowPerShare from '@/components/CashFlowPerShare.vue';
import DividendYield from '@/components/fundamentalAdvance/DividendYield.vue';
import DividendPayout from '../components/fundamentalAdvance/DividendPayout.vue';
import RetentionRatio from '../components/fundamentalAdvance/RetentionRatio.vue';
import DividendCoverage from '@/components/fundamentalAdvance/DIvidendCoverage.vue';

import { type BreadcrumbItem } from '@/types';

const breadcrumbs: BreadcrumbItem[] = [
  { title: 'Dividends', href: '/dividends' },
];

const selectedSector = ref('37');
const sectors = ref<{ sector_id: number; sector_description: string; symbols: string[] }[]>([]);

const activeTab = ref('Cash Flow/Share');

const tabs = [
    { name: "Cash Flow/Share", component: CashFlowPerShare },
    { name: "Dividend Yeild", component: DividendYield },
    { name: "Dividend Payout", component: DividendPayout },
    { name: "Retention Ratio", component: RetentionRatio },
    { name: "Dividend Coverage", component: DividendCoverage },
];

const fetchSectors = async () => {
  try {
    const response = await fetch('https://pro.laganisutra.com/api/sector-symbols');
    const data = await response.json();
    sectors.value = data;
  } catch (error) {
    console.error('Error fetching sectors:', error);
  }
};

onMounted(() => {
  fetchSectors();
});
</script>

<template>
  <Head title="Dividends" />
  <AppLayout :breadcrumbs="breadcrumbs">
    <div class="p-6">
      <h1 class="mb-6 text-2xl font-bold">Dividends</h1>

      <!-- Sector Select -->
      <div class="mb-4">
        <label class="block mb-2 font-medium">Select Sector:</label>
        <select v-model="selectedSector" class="border rounded px-3 py-2">
          <option value="all">All Sectors</option>
          <option
            v-for="sector in sectors"
            :key="sector.sector_id"
            :value="sector.sector_id"
            :data-name="sector.sector_description"
          >
            {{ sector.sector_description }}
          </option>
        </select>
      </div>

      <!-- Tab Buttons -->
      <div class="mb-4 flex gap-2 border-b pb-2">
        <button
          v-for="tab in tabs"
          :key="tab.name"
          :class="[
            'px-4 py-2 font-semibold rounded-t-lg',
            activeTab === tab.name ? 'bg-blue-500 text-white' : 'bg-gray-100'
          ]"
          @click="activeTab = tab.name"
        >
          {{ tab.name }}
        </button>
      </div>

      <!-- Active Component -->
      <component
        :is="tabs.find(tab => tab.name === activeTab)?.component"
        :selected-sector="selectedSector"
      />
    </div>
  </AppLayout>
</template>