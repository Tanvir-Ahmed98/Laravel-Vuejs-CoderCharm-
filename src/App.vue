<template>
    <div class="phone-call-log-app">
    <div class="filter-section">
      <label for="statusFilter">Status:</label>
      <select v-model="statusFilter" @change="applyFilter">
        <option value="">All</option>
        <option value="In-call">In-call</option>
        <option value="Hold">Hold</option>
        <option value="Call back">Call back</option>
        <option value="Do not call">Do not call</option>
      </select>

      <label for="startDate">Start Date:</label>
      <input type="date" v-model="startDate" @change="applyFilter">

      <label for="endDate">End Date:</label>
      <input type="date" v-model="endDate" @change="applyFilter">
    </div>

   
    <div class="chart-container">
      <canvas ref="lineChart" width="800" height="300"></canvas>
    </div>

<div class="phone-call-log-table">
  <table>
    <thead>
      <tr class="grid-header">
        <th>Call Date</th>
        <th>Phone Number</th>
        <th>Call Duration<br><span>(In Second)</span></th>
        <th>Status</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="log in paginatedLogs" :key="log.id" class="log-item">
        <td>{{ formatDate(log.call_date) }}</td>
        <td>{{ log.phone_number }}</td>
        <td>{{ log.call_duration }}</td>
        <td :class="getStatusColor(log.status)">{{ log.status }}</td>
      </tr>
    </tbody>
  </table>
</div>
    <div class="pagination-section">
      <button @click="prevPage" :disabled="currentPage === 1">Previous</button>
      <span>{{ currentPage }} / {{ totalPages }}</span>
      <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Chart from 'chart.js';
import 'chartjs-adapter-dayjs';

export default {
  data() {
    return {
      phoneCallLogs: [],
      displayedLogs: [],
      currentPage: 1,
      itemsPerPage: 25,
      statusFilter: '',
      startDate: null,
      endDate: null,
      lineChart: null,
    };
  },
  mounted() {
    this.fetchPhoneCallLogs();
    this.initLineChart();
  },
  methods: {
    async fetchPhoneCallLogs() {
      try {
        const response = await axios.get('http://127.0.0.1:8000/api/phone-call-logs');
        this.phoneCallLogs = response.data;
        this.applyFilter(); 
      } catch (error) {
        console.error('Error fetching data:', error);
       
      }
    },
initLineChart() {
  const ctx = this.$refs.lineChart.getContext('2d');
  this.lineChart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: [], // Dates will be added dynamically
      datasets: [
        {
          label: 'Number of Calls',
          borderColor: 'blue',
          borderWidth: 2,
          data: [], // Call count will be added dynamically
        },
      ],
    },
    options: {
      plugins: {
        legend: {
          display: false,
        },
      },
      scales: {
        x: {
          type: 'time',
          time: {
            unit: 'day',
          },
          title: {
            display: true,
            text: 'Date',
          },
        },
        y: {
          beginAtZero: true,
          title: {
            display: true,
            text: 'Number of Calls',
          },
        },
      },
    },
  });
},


  updateLineChart() {
  // Get the counts of calls for each date
  const counts = this.getCountsByDate();

  // Ensure that counts and labels are defined
  if (counts && this.lineChart.data.labels) {
    // Update the chart data
    this.lineChart.data.labels = Object.keys(counts);
    this.lineChart.data.datasets[0].data = Object.values(counts);

    // Update the chart
    this.lineChart.update();
  }
},
    getCountsByDate() {
      const counts = {};
      this.displayedLogs.forEach(log => {
        const date = new Date(log.call_date).toLocaleDateString();
        counts[date] = (counts[date] || 0) + 1;
      });
      return counts;
    },
    applyFilter() {
      this.displayedLogs = this.phoneCallLogs.filter(log => {
        const statusFilter = this.statusFilter === '' || log.status === this.statusFilter;
        const dateFilter = this.filterByDate(log.call_date);
        return statusFilter && dateFilter;
      });

      this.currentPage = 1;
      this.updateLineChart();
    },
    filterByDate(callDate) {
      if (this.startDate && this.endDate) {
        return callDate >= this.startDate && callDate <= this.endDate;
      } else {
        return true;
      }
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
    formatDate(dateString) {
      const options = { year: 'numeric', month: 'long', day: 'numeric' };
      return new Date(dateString).toLocaleDateString(undefined, options);
    },
  getStatusColor(status) {
  switch (status) {
    case 'In-call':
      return 'status-In-call';
    case 'Hold':
      return 'status-Hold';
    case 'Call back':
      return 'status-Call-back';
    case 'Do not call':
      return 'status-Do-not-call';
    default:
      return ''; // Default class if the status is not recognized
  }
},
  },
  computed: {
    totalPages() {
      return Math.ceil(this.displayedLogs.length / this.itemsPerPage);
    },
    paginatedLogs() {
      const startIndex = (this.currentPage - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      return this.displayedLogs.slice(startIndex, endIndex);
    },
  },
};
</script>

<style scoped>

.filter-section {
  display: flex;
  gap: 20px;
  align-items: center;
  margin-bottom: 20px;
}

.filter-input {
  display: flex;
  flex-direction: column;
}

label {
  font-weight: bold;
  margin-bottom: 5px;
  color: #333;
  size:20%;
}

select, input {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 16px;
  width: 200px;
  transition: border-color 0.3s ease-in-out;
}

select:focus, input:focus {
  outline: none;
  border-color: #4CAF50;
}


.search-btn {
  padding: 10px 15px;
  background-color: #4CAF50;
  color: #fff;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s ease-in-out;
}

.search-btn:hover {
  background-color: #45a049;
}


.phone-call-log-app {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.filter-section {
  margin-bottom: 20px;
}

label {
  margin-right: 10px;
}

.chart-container {
  margin-bottom: 20px;
}

.phone-call-log-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}

.grid-header {
  font-weight: bold;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}

.log-item {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  border: 1px solid #ddd;
  padding: 10px;
}

.pagination-section {
  margin-top: 20px;
  display: flex;
  justify-content: space-between;
}

button {
  padding: 8px 16px;
  cursor: pointer;
  background-color: #4caf50;
  color: #fff;
  border: none;
  border-radius: 4px;
  font-size: 14px;
}

button[disabled] {
  background-color: #ccc;
  cursor: not-allowed;
}


.status {
  font-weight: bold;
  padding: 6px;
  border-radius: 20px;
}

.status-In-call {
  background-color: #4caf50;
  text:inline;
  text-align: center;
  border-radius: 20px;
  color: #fff;
}

.status-Hold {
  background-color: #ffc107;
 
   text-align: center;
  border-radius: 20px;
  color: #000;
}

.status-Call-back {
  background-color: #2196f3;
   text-align: center;
  border-radius: 20px;
  color: #fff;
}

.status-Do-not-call {
  background-color: #f44336;
   text-align: center;
  border-radius: 20px;
  color: #fff;
}
td{
 text-align: center;

}
</style>

