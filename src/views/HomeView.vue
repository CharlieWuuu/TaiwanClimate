<template>
    <img
        src="../../public/img/icons/climate_sample.png"
        alt=""
        style="width: 300px; margin: auto"
    />
    <div class="weatherChart_group_container">
        <div
            class="weatherChart_container"
            v-for="(l, index) in location"
            :key="index"
        >
            <div class="title_container">
                <p>
                    {{ l.station.StationName }}
                    ｜
                    {{ l.station.StationName == '玉山' ? '高地氣候' : chartData_Arr[index].datasets[1].data[0] < 18 ? '副熱帶季風氣候' : '熱帶季風氣候' }}
                </p>
            </div>
            <div
                class="chart-container"
                style="position: relative; width: 310px; height: 350px"
            >
                <Bar
                    v-if="loaded"
                    class="Chart"
                    :data="chartData_Arr[index]"
                    :options="chartOptions"
                    style="width: 100%; height: 100%"
                />
            </div>
        </div>
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
        loaded: false,
        location: {},
        chartData_Arr: [],

        chartOptions: {
            responsive: true,
            scales: {
                bottom: {
                    grid: {
                        drawTicks: false,
                        color: '#f0f0f0',
                    },
                },
                left: {
                    position: 'left',
                    min: -6,
                    max: 36,
                    ticks: {
                        stepSize: 6,
                    },
                    grid: {
                        drawTicks: false,
                        color: '#f0f0f0',
                    },
                },
                right: {
                    position: 'right',
                    min: 0,
                    max: 840,
                    ticks: {
                        stepSize: 120,
                    },
                    grid: {
                        drawTicks: false,
                        color: '#f0f0f0',
                    },
                },
            },
        },
        chartKey: 0, // 添加一個鍵，用於強制重新渲染圖表
    }),
    async mounted() {
        try {
            await this.getClimateData();
        } catch (e) {
            console.error(e);
        }
    },

    methods: {
        async getClimateData() {
            this.loaded = false;
            const response = await fetch('https://opendata.cwa.gov.tw/api/v1/rest/datastore/C-B0027-001?Authorization=CWB-C67AAE13-37AA-4F9D-892F-E25483690887');
            if (!response.ok) {
                throw new Error('Failed to fetch data');
            }

            const data = await response.json();
            const location = data.records.data.surfaceObs.location;
            this.location = location;
            for (var i = 0; i < location.length; i++) {
                const stationObsStatistics = location[i].stationObsStatistics;
                const Precipitation_Arr = stationObsStatistics.Precipitation.monthly.map((element) => parseFloat(element.Accumulation));
                const AirTemperature_Arr = stationObsStatistics.AirTemperature.monthly.map((element) => parseFloat(element.Mean));

                let chartData = {
                    labels: ['1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12'],
                    datasets: [
                        {
                            type: 'line',
                            label: '氣溫',
                            backgroundColor: '#ee2f2a',
                            borderColor: '#ee2f2a',
                            pointRadius: 0,
                            data: AirTemperature_Arr,
                            order: 1,
                            xAxisID: 'bottom',
                            yAxisID: 'left',
                        },
                        {
                            type: 'bar',
                            label: '降水量',
                            backgroundColor: '#abe1fa',
                            borderColor: '#00aeef',
                            borderWidth: 1,
                            barThickness: 22,
                            data: Precipitation_Arr,
                            order: 2,
                            xAxisID: 'bottom',
                            yAxisID: 'right',
                        },
                    ],
                };
                this.chartData_Arr.push(chartData);
            }
            this.loaded = true; // 在數據加載後設置為true
        },
    },
};
</script>

<style lang="scss">
.page {
    display: flex;
    flex-direction: column;
    max-width: 100%;
    .weatherChart_group_container {
        height: fit-content;
        display: flex;
        gap: 20px;
        flex-wrap: wrap;
        justify-content: center;

        .weatherChart_container {
            background-color: #eff9fe;
            .title_container {
                display: flex;
                width: 100%;
                justify-content: center;
                align-items: center;
                gap: 8px;
                p {
                    font-size: 20px;
                }
            }
        }
    }
}
</style>
