<script setup>
import { Line } from 'vue-chartjs';
import { LinearScale, Chart, PointElement, LineElement } from 'chart.js';
import { ref, computed } from 'vue';
Chart.register(LinearScale, PointElement, LineElement);
Chart.defaults.backgroundColor = '#0800FF';
Chart.defaults.borderColor = '#AAAAAA';
Chart.defaults.color = '#FFFFFF';

const props = defineProps({
    model: {type: String, default: 'mm1'}
});

function mm1(rho) {
    return rho / (1-rho);
}

function kingman(rho, c_s, c_a) {
    return (Math.pow(c_a, 2) + Math.pow(c_s, 2))*mm1(rho)/2;
}

const start = 0;
const end = 1;
const steps = 100;
const delta = (end - start)/steps

const x = Array.from({length: steps}, (_, i) => i*delta + start);
const model = ref(props.model);
const variance = ref(1);
const chart = ref(null);


const data = computed(function() {
    let y;
    switch(model.value) {
        case "mm1":
            y = x.map(mm1);
            break;
        case "kingman":
            y = x.map((x) => kingman(x, variance.value, 1));
            break;
    }
    return {
        labels: x,
        datasets: [{label: model.value, data: y, borderColor: '#9399FF'}]
    }
});
const options = {
    title: 'Queue latency',
    elements: {
        point: {pointStyle: false}
    },
    scales: {
        y: {
            title: {text: 'Latency/Queue size', display: true},
            ticks: {display: false},
            min: 0,
            max: 100
        },
        x: {
            title: {text: 'Utilization', display: true},
            type: 'linear',
            min: start,
            max: end
        },
    }
}

</script>

<style scoped>
.chart {
    width: 80vh;
    margin: auto;
}
.container {
    height: fit-content;
    width: fit-content;
    display: block;
}
input {
    margin-left: 5px;
}
</style>

<template>
    <div class="container">
        <div class="chart">
            <Line ref="chart" :data="data" :options="options"></Line>
        </div>
    
        <template v-if="model == 'kingman'">
            <label for="variance">Variance</label><input label="Variance" type="range" name="variance" id="" min="1" max="10" v-model="variance">
        </template>
    </div>
</template>

