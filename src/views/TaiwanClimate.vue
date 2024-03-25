<template>
    <h1>5. 串接資料與圖表</h1>
    <div class="select_container">
        <p>觀測站：</p>
        <select
            v-model="selectedStationIndex"
            v-on:change="getClimateData(selectedStationIndex)"
        >
            <option
                v-for="(station, index) in Stations"
                :value="index"
                :key="index"
            >
                {{ station.station.StationName }}
            </option>
        </select>
        <p>
            {{ Stations_Name == '玉山' ? '高地氣候' : chartData.datasets[1].data[0] < 18 ? '副熱帶季風' : '熱帶季風' }}
        </p>
    </div>
    <div
        class="chart-container"
        style="position: relative; width: 350px; height: 500px"
    >
        <Bar
            v-if="loaded"
            class="Chart"
            :data="chartData"
            :options="chartOptions"
            style="width: 100%; height: 100%"
        />
    </div>
</template>

<script>
import { Bar } from 'vue-chartjs';

import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale, PointElement, LineController, LineElement } from 'chart.js';

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale, PointElement, LineController, LineElement);

export default {
    name: 'BarChart',
    components: { Bar },

    data: () => ({
        selectedStationIndex: 0,
        loaded: false,
        Stations: {},
        chartData: {
            labels: ['1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12'],
            datasets: [
                {
                    type: 'bar',
                    label: '降水量',
                    backgroundColor: '#46A3FF',
                    data: [40, 20, 78, 60, 128, 170, 153, 112, 80, 72, 63, 12],
                    order: 2,
                    yAxisID: 'left',
                },
                {
                    type: 'line',
                    label: '氣溫',
                    backgroundColor: '#FF5151',
                    borderColor: '#FF5151',
                    data: [20, 26, 28, 30, 34, 35, 36, 34, 28, 25, 23, 21],
                    order: 1,
                    yAxisID: 'right',
                },
            ],
        },
        chartOptions: {
            responsive: true,
            scales: {
                left: {
                    position: 'left',
                    suggestedMin: 0,
                    suggestedMax: 1000,
                    ticks: {
                        stepSize: 100,
                    },
                },
                right: {
                    position: 'right',
                    suggestedMin: -10,
                    suggestedMax: 40,
                    ticks: {
                        stepSize: 10,
                    },
                },
            },
        },
        chartKey: 0, // 添加一個鍵，用於強制重新渲染圖表
    }),
    async mounted() {
        try {
            await this.getClimateData(22);
        } catch (e) {
            console.error(e);
        }
    },

    methods: {
        async getClimateData(i) {
            this.loaded = false;
            const response = await fetch('https://opendata.cwa.gov.tw/api/v1/rest/datastore/C-B0027-001?Authorization=CWB-C67AAE13-37AA-4F9D-892F-E25483690887');
            if (!response.ok) {
                throw new Error('Failed to fetch data');
            }

            const data = await response.json();
            this.Stations = data.records.data.surfaceObs.location;

            const targetData = data.records.data.surfaceObs.location[i];

            const station = targetData.station;
            const stationObsStatistics = targetData.stationObsStatistics;
            const stationsName = station.StationName;
            this.Stations_Name = stationsName;

            const Precipitation_Arr = stationObsStatistics.Precipitation.monthly.map((element) => parseFloat(element.Accumulation));
            this.chartData.datasets[0].data = Precipitation_Arr;

            const AirTemperature_Arr = stationObsStatistics.AirTemperature.monthly.map((element) => parseFloat(element.Mean));
            this.chartData.datasets[1].data = AirTemperature_Arr;
            console.log(targetData);
            this.loaded = true; // 在數據加載後設置為true
        },
    },
};
</script>

<style lang="scss">
.select_container {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 40px;
    p {
        font-size: 20px;
    }
    select {
        border: none;
        font-size: 20px;
    }
}
</style>
