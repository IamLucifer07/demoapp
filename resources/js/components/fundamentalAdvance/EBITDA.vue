<template>
    <div class="total-assets-container">
        <!-- Sector Selection -->
        <div class="sector-selection">
            <div class="card">
                <div class="form-group">
                    <label for="sector">Select Sector:</label>
                    <v-select
                        v-model="selectedSector"
                        :options="sectors"
                        @update:modelValue="fetchTotalAssetsData"
                        placeholder="Select a sector"
                        :reduce="(sector) => sector.id"
                        label="sectorDescription"
                        :style="{ width: '30%' }"
                    ></v-select>
                </div>

                <!-- Using the ResultTable component -->
                <ResultTable
                    :columns="tableColumns"
                    :results="results"
                    :errors="errors"
                    :isLoading="isLoading"
                    :showResults="showResults"
                />
            </div>
        </div>
    </div>
</template>

<script>
import axios from "axios";
import vSelect from "vue-select";
import "vue-select/dist/vue-select.css";
import ResultTable from "./ResultTable.vue";

export default {
    name: "EBITDA",
    components: {
        vSelect,
        ResultTable,
    },
    data() {
        return {
            sectors: [],
            selectedSector: null,
            results: [],
            errors: [],
            isLoading: false,
            showResults: false,
           tableColumns: [
            { key: 'symbol', label: 'Symbol' },
            { key: 'fiscal_year', label: 'Fiscal Year' },
            { key: 'quarter', label: 'Quarter' },
            { key: 'market_capitalization', label: 'Market Capitalization' },
            { key: 'total_debt', label: 'Total Debt' },
            { key: 'cash_and_cash_equivalent', label: 'Cash Equivalent' },
            { key: 'ev_ebitda', label: 'EV/EBITDA' },
          ],
        };
    },
    async created() {
        await this.fetchSectors();
        if (this.sectors.length > 0) {
            this.selectedSector = this.sectors[0].id;
            await this.fetchTotalAssetsData();
        }
    },
    methods: {
        async fetchSectors() {
            try {
                const allSectors = getSectors(); // assuming this returns array
                this.sectors = allSectors;
                // console.log("sectors", allSectors);
            } catch (error) {
                console.error("Error fetching sectors:", error);
                this.errors.push(
                    "Failed to load sectors. Please try again later."
                );
            }
        },

        async fetchTotalAssetsData() {
            if (!this.selectedSector) return;

            this.isLoading = true;
            this.results = [];
            this.errors = [];
            this.showResults = true;

            try {
                const response = await axios.get("/api/EvEbitda", {
                    params: {
                        sector: this.selectedSector,
                    },
                });

                if (Array.isArray(response.data)) {
                    this.results = response.data;
                } else if (response.data && response.data.data) {
                    this.results = response.data.data;
                    this.errors = response.data.errors || [];
                }
            } catch (error) {
                console.error("Error fetching total assets data:", error);
                this.errors.push(
                    "Failed to load total assets data. Please try again later."
                );
            } finally {
                this.isLoading = false;
            }
        },
    },
};
</script>