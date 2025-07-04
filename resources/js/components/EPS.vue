<template>
  <div class="financial-metrics-container">
    
    <!-- Loading State -->
    <div v-if="isLoading" class="text-center py-4">
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
      <p class="mt-2">Loading financial data...</p>
    </div>

    <!-- Error State -->
    <div v-else-if="errors.length > 0" class="alert alert-danger">
      <h5>Errors occurred:</h5>
      <ul class="mb-0">
        <li v-for="error in errors" :key="error">{{ error }}</li>
      </ul>
    </div>

    <!-- Results Table -->
    <div v-if="showResults && results.length > 0" class="table-responsive">
      <table class="custom-table">
        <thead>
          <tr>
            <th v-for="column in tableColumns" :key="column.key">
              {{ column.label }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr 
            v-for="(result, index) in results" 
            :key="index"
            :class="getRowClass(result)"
            :title="getRowTooltip(result)"
          >
            <td v-for="column in tableColumns" :key="column.key">
              <template v-if="column.key === 'recommendation'">
                <span 
                  class="recommendation-badge"
                  :class="getRecommendationClass(getRecommendation(result))"
                  :title="getRecommendationTooltip(getRecommendation(result))"
                >
                  {{ getRecommendation(result) }}
                </span>
              </template>
              <template v-else-if="isNumericField(column.key)">
                {{ formatNumber(result[column.key]) }}
              </template>
              <template v-else>
                {{ result[column.key] || '-' }}
              </template>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- No Results State -->
    <div v-else-if="showResults && results.length === 0" class="text-center py-4">
      <p class="text-muted">No financial data available for the selected sector.</p>
    </div>
  
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'EPS',
  
  props: {
    selectedSector: {
      type: [String, Number],
      default: null
    }
  },
  
  data() {
    return {
      results: [],
      errors: [],
      isLoading: false,
      showResults: false,
      tableColumns: [
        { key: 'symbol', label: 'Symbol' },
        { key: 'fiscal_year', label: 'Fiscal Year' },
        { key: 'quarter', label: 'Quarter' },
        { key: 'eps', label: 'EPS' },
        { key: 'roe', label: 'ROE (%)' },
        { key: 'pb_ratio', label: 'P/B Ratio' },
        { key: 'pe_ratio', label: 'P/E Ratio' },
        { key: 'net_income', label: 'Net Income' },
        { key: 'revenue', label: 'Revenue' },
        { key: 'gross_profit', label: 'Gross Profit' },
        { key: 'recommendation', label: 'Recommendation' }
      ],
      sectorMap: {
        "37": "Commercial Banks",
        "44": "Development Banks",
        "45": "Finance",
        "49": "Microfinance",
        "43": "Non Life Insurance",
        "50": "Life Insurance",
        "41": "Hydro Power",
        "39": "Hotels And Tourism",
        "38": "Manufacturing And Processing",
        "42": "Tradings",
        "67": "Investment",
        "40": "Others"
      },
      epsRatingRanges: {
      "Commercial Banks": {
        best: 40, better: [25, 40], good: [15, 25], neutral: [10, 15], weak: [5, 10], worst: 5
      },
      "Hydro Power": {
        best: 30, better: [20, 30], good: [12, 20], neutral: [8, 12], weak: [4, 8], worst: 4
      },
      "Non Life Insurance": {
        best: 25, better: [15, 25], good: [10, 15], neutral: [6, 10], weak: [3, 6], worst: 3
      },
      "Life Insurance": {
        best: 20, better: [12, 20], good: [8, 12], neutral: [5, 8], weak: [2, 5], worst: 2
      },
      "Manufacturing And Processing": {
        best: 15, better: [10, 15], good: [7, 10], neutral: [4, 7], weak: [2, 4], worst: 2
      },
      "Tradings": { 
        best: 12, better: [8, 12], good: [5, 8], neutral: [3, 5], weak: [1, 3], worst: 1
      },
      "Investment": { 
        best: 12, better: [8, 12], good: [5, 8], neutral: [3, 5], weak: [1, 3], worst: 1
      },
      "Hotels And Tourism": {
        best: 10, better: [6, 10], good: [4, 6], neutral: [2, 4], weak: [1, 2], worst: 1
      },
      "Development Banks": {
        best: 18, better: [12, 18], good: [8, 12], neutral: [5, 8], weak: [3, 5], worst: 3
      },
      "Microfinance": {
        best: 15, better: [10, 15], good: [6, 10], neutral: [4, 6], weak: [2, 4], worst: 2
      },
      "Finance": {
        best: 15, better: [10, 15], good: [6, 10], neutral: [4, 6], weak: [2, 4], worst: 2
      },
      "Others": {
        best: 10, better: [7, 10], good: [5, 7], neutral: [3, 5], weak: [1, 3], worst: 1
      }
      },
      recommendations: {
        'Best': {
          interpretation: 'Exceptional profitability. Industry leaders with strong competitive advantages.',
          class: 'recommendation-best'
        },
        'Better': {
          interpretation: 'Above-average profitability. Well-managed companies with growth potential.',
          class: 'recommendation-better'
        },
        'Good': {
          interpretation: 'Decent profitability. Stable but not outstanding performers.',
          class: 'recommendation-good'
        },
        'Neutral': {
          interpretation: 'Marginal profitability. Needs monitoring for improvement.',
          class: 'recommendation-neutral'
        },
        'Weak': {
          interpretation: 'Concerning profitability. May indicate operational challenges.',
          class: 'recommendation-weak'
        },
        'Worst': {
          interpretation: 'Dangerous zone. Often loss-making or near-zero profitability.',
          class: 'recommendation-worst'
        }
      }
    };
  },
  
  watch: {
    selectedSector: {
      handler() {
        if (this.selectedSector) {
          this.getFinancialMetrics();
        }
      },
      immediate: true
    }
  },
  
  methods: {
    async getFinancialMetrics() {
      this.isLoading = true;
      this.results = [];
      this.errors = [];
      this.showResults = true;

      try {
        // const sector = (this.selectedSector ?? '').toString().trim() || 'all';
        const sectorDescription = this.sectorMap[this.selectedSector] || 'all';
        const url = `https://laganisutra.com/api/database-values?sector=${encodeURIComponent(sectorDescription)}`;

        const response = await axios.get(url);

        if (Array.isArray(response.data)) {
          this.results = response.data;
        } else if (response.data && response.data.errors) {
          this.errors = response.data.errors;
        } else {
          this.errors.push('Unexpected response format.');
        }
      } catch (error) {
        if (error.response) {
          this.errors.push(`API error: ${error.response.data.message || 'Unknown error.'}`);
        } else {
          this.errors.push('Network or CORS error. Check the console for more info.');
        }
      } finally {
        this.isLoading = false;
      }
    },
    
    formatNumber(value) {
      return Number(value).toFixed(2);
    },
    
    isNumericField(key) {
      const numericFields = ['eps', 'roe', 'pb_ratio', 'pe_ratio', 'net_income', 'revenue', 'gross_profit'];
      return numericFields.includes(key);
    },
    
     getRecommendation(row) {
        const sector = String(this.selectedSector || "").trim();

        const epsValue = parseFloat(row.eps);

        // === EPS classification ===
        const epsRanges = this.epsRatingRanges[sector] || this.epsRatingRanges['Others'];
        let epsRating = '';
        if (epsRanges && !isNaN(epsValue)) {
          if (epsValue >= epsRanges.best) epsRating = 'Best';
          else if (epsValue >= epsRanges.better[0]) epsRating = 'Better';
          else if (epsValue >= epsRanges.good[0]) epsRating = 'Good';
          else if (epsValue >= epsRanges.neutral[0]) epsRating = 'Neutral';
          else if (epsValue >= epsRanges.weak[0]) epsRating = 'Weak';
          else epsRating = 'Worst';
        }

        return epsRating || 'N/A';

      },
    
    getRecommendationClass(recommendation) {
      return this.recommendations[recommendation]?.class || 'recommendation-neutral';
    },
    
    getRecommendationTooltip(recommendation) {
      return this.recommendations[recommendation]?.interpretation || '';
    },
    
    getRowClass(result) {
      const recommendation = this.getRecommendation(result);
      return this.getRecommendationClass(recommendation);
    },
    
    getRowTooltip(result) {
      const recommendation = this.getRecommendation(result);
      return this.getRecommendationTooltip(recommendation);
    }
  }
};
</script>

<style scoped src="resources/css/fundamental.css"></style>