<script setup lang="ts">
import { Scatter } from 'vue-chartjs'
import { Chart as ChartJS, Tooltip, Legend, PointElement, LinearScale } from 'chart.js'
import zoomPlugin from 'chartjs-plugin-zoom'

ChartJS.register(Tooltip, Legend, PointElement, LinearScale, zoomPlugin)

interface DataPoint {
  id: number
  propertyId: number
  month: number
  cost: number
}

// Dummy data
const dummyData: DataPoint[] = [
  { id: 1, propertyId: 1, month: 1, cost: 100000 },
  { id: 2, propertyId: 1, month: 2, cost: 105000 },
  { id: 3, propertyId: 1, month: 3, cost: 102000 },
  { id: 4, propertyId: 1, month: 4, cost: 110000 },
  { id: 5, propertyId: 1, month: 5, cost: 108000 },
  { id: 6, propertyId: 1, month: 6, cost: 115000 },
  { id: 7, propertyId: 1, month: 7, cost: 112000 },
  { id: 8, propertyId: 1, month: 8, cost: 118000 },
  { id: 9, propertyId: 1, month: 2, cost: 120000 },
  { id: 10, propertyId: 1, month: 10, cost: 125000 },
  { id: 11, propertyId: 1, month: 11, cost: 122000 },
  { id: 12, propertyId: 1, month: 12, cost: 130000 },
  { id: 13, propertyId: 2, month: 1, cost: 80000 },
  { id: 14, propertyId: 2, month: 3, cost: 85000 },
  { id: 15, propertyId: 2, month: 5, cost: 82000 },
  { id: 16, propertyId: 2, month: 7, cost: 88000 },
  { id: 17, propertyId: 2, month: 9, cost: 90000 },
  { id: 18, propertyId: 2, month: 11, cost: 95000 },
  { id: 19, propertyId: 3, month: 2, cost: 120000 },
  { id: 20, propertyId: 3, month: 4, cost: 125000 },
  { id: 21, propertyId: 3, month: 6, cost: 122000 },
  { id: 22, propertyId: 3, month: 8, cost: 130000 },
  { id: 23, propertyId: 3, month: 10, cost: 135000 },
  { id: 24, propertyId: 3, month: 12, cost: 140000 },
]

const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']

// Group data by propertyId
const groupedData: Record<number, { x: number; y: number }[]> = {}
for (const item of dummyData) {
  groupedData[item.propertyId] = groupedData[item.propertyId] || []
  // Add small jitter to x to avoid overlapping points
  const jitterX = (Math.random() - 0.5) * 0.2  // Random offset between -0.1 and 0.1
  groupedData[item.propertyId]!.push({ x: item.month + jitterX, y: item.cost })
}

const datasets: any[] = []
for (const propertyId in groupedData) {
  datasets.push({
    label: `Property ${propertyId}`,
    data: groupedData[Number(propertyId)],
    backgroundColor: `hsl(${Math.random() * 360}, 70%, 50%)`,
    pointRadius: 5,
    pointHoverRadius: 10,
    pointHoverBorderWidth: 3,
    pointHoverBorderColor: 'rgba(255,255,0,0.8)',
  })
}

const chartData = {
  datasets,
}

const chartOptions = {
  responsive: true,
  hover: {
    mode: 'dataset' as const,
  },
  plugins: {
    legend: {
      position: 'top' as const,
    },
    tooltip: {
      callbacks: {
        label: (context: any) => {
          const month = Math.round(context.parsed.x)  // Round to nearest month
          const cost = context.parsed.y
          return `${context.dataset.label}: $${cost} (Month: ${months[month - 1] || month})`
        },
      },
    },
    zoom: {
      zoom: {
        wheel: {
          enabled: true,
        },
        pinch: {
          enabled: true,
        },
        mode: 'xy' as const,
      },
      pan: {
        enabled: true,
        mode: 'xy' as const,
      },
    },
  },
  scales: {
    x: {
      type: 'linear' as const,
      ticks: {
        callback: (tickValue: string | number) => {
          const num = typeof tickValue === 'number' ? tickValue : Number.parseFloat(tickValue.toString())
          return months[num - 1] || tickValue
        },
      },
      title: {
        display: true,
        text: 'Month',
      },
    },
    y: {
      title: {
        display: true,
        text: 'Cost ($)',
      },
    },
  },
}
</script>

<template>
  <div class="max-w-6xl mt-4 mx-auto p-6 bg-white rounded-lg shadow-lg">
    <div class="mb-6">
      <h1 class="text-3xl font-bold text-gray-800 mb-2">Property Cost Scatter Plot</h1>
      <p class="text-gray-600">Hover over points to highlight all data for the same property. Use zoom and pan for detailed exploration.</p>
    </div>
    <div class="bg-gray-50 p-4 rounded-md border">
      <Scatter :data="chartData" :options="chartOptions" class="w-full h-96" />
    </div>
    <div class="mt-4 text-sm text-gray-500">
      <p><strong>Note:</strong> Points are slightly jittered to avoid overlaps. Tooltips show exact values.</p>
    </div>
  </div>
</template>
