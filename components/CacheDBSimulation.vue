<template>
    <div class="container">
        <div class="w-full columns-2">
            <div>
                <label for="cache-ratio">
                    Cache hit/miss ratio
                    <input type="range" name="cache-ratio" id="cache-ratio" min="0.8" max="0.99" step="0.01" v-model="cacheRatio">
                </label><span class="ml-2">{{ cacheRatio * 100 }}%</span>
            </div>
            <div>
                <label for="cache-latency">
                    Mean cache latency
                    <input type="range" name="cache-latency" id="cache-latency" min="1" max="20" v-model.number="cacheLatency">{{ cacheLatency }}ms
                </label>
            </div>
            <div>
                <label for="cache-variance">
                    Cache standard deviation
                    <input type="range" name="cache-variance" id="cache-variance" min="0.2" max="4" step="0.2" v-model.number="cacheSTDDev">{{ cacheSTDDev }}ms
                </label>
            </div>
            <div>
                <label for="db-latency">
                    DB latency
                    <input type="range" name="db-latency" id="db-latency" min="1" max="150" v-model.number="dbLatency">{{ dbLatency }}ms
                </label>
            </div>
            <div>
                <label for="db-variance">
                    DB standard deviation
                    <input type="range" name="db-variance" id="db-variance" min="1" max="50" v-model.number="dbSTDDev">{{ dbSTDDev }}ms
                </label>
            </div>
            <div>
                <label for="arrival-rate">
                    Mean load [req/sec]
                    <input type="range" name="arrival-rate" id="arrival-rate" min="50" max="500" v-model.number="arrivalRate">{{ arrivalRate }} req/sec
                </label>
            </div>
            <button type="button" :disabled="running" class="disabled:opacity-50 my-2 px-2 py-1 bg-blue-500 hover:bg-blue-300 rounded text-white font-bold" @click="start">Run simulation</button>
        </div>
        <div v-if="data" class="results w-full mt-6">
            <div class="overflow-auto h-80">
                <div v-html="data.populationTable"></div>
                <div v-html="data.serverHistogram"></div>
                <div v-html="data.dbHistogram"></div>
                <div v-html="data.cacheHistogram"></div>
            </div>
        </div>
    </div>
</template>

<style>
.ss-histogram rect {
    fill: rgb(145, 145, 255);
}

.ss-histogram rect.avg {
    fill: rgb(145, 145, 255);
}

.ss-histogram text {
    fill: white;
}

</style>

<script setup lang="ts">
import {ref} from 'vue';
import {
    Exponential,
    LogNormal,
    Simulation,
    Queue,
    Tally,
    Entity,
} from 'simscript';


class CacheAndDB extends Simulation {
    serverThreads;
    qCacheService = new Queue('cacheService', 1);
    qDBService = new Queue('DBService', 1);
    server = new Queue('Server');
    qCache = new Queue('Cache');
    qDB = new Queue('DB');
    dbService: LogNormal;
    cacheService: LogNormal;
    interArrival;
    cacheRatio;
    tally = new Tally();

    constructor(threads, arrivalRate,
        cacheRatio, cacheLatency, cacheSTDDev,
        dbLatency, dbSTDDev
    ) {
        super();
        this.serverThreads = new Queue('ServerThreads', threads);
        this.cacheService = new LogNormal(cacheLatency, cacheSTDDev);
        this.dbService = new LogNormal(dbLatency, dbSTDDev);
        this.interArrival = new Exponential(1000/arrivalRate)
        this.cacheRatio = cacheRatio;
    }

    onStarting() {
        this.qCache.grossPop.setHistogramParameters(2);
        this.qCache.grossDwell.setHistogramParameters(2);
        this.qDB.grossPop.setHistogramParameters(2);
        this.qDB.grossDwell.setHistogramParameters(Math.max((this.dbService.mean + this.dbService.std*3) / 10, 20));
        this.server.grossDwell.setHistogramParameters(Math.max((this.dbService.mean + this.dbService.std*3)/10, 20));
        this.generateEntities(Task, this.interArrival, 10000);
    }
}

class Task extends Entity<CacheAndDB> {
    async script() {
        let sim = this.simulation;
        this.enterQueueImmediately(sim.server);
        await this.enterQueue(sim.serverThreads);
        await this.enterQueue(sim.qCache);
        await this.enterQueue(sim.qCacheService);
        this.leaveQueue(sim.qCache);
        await this.delay(sim.cacheService.sample());
        this.leaveQueue(sim.qCacheService);

        if (Math.random() > sim.cacheRatio) {
            await this.enterQueue(sim.qDB);
            await this.enterQueue(sim.qDBService);
            this.leaveQueue(sim.qDB);
            await this.delay(sim.dbService.sample());
            this.leaveQueue(sim.qDBService);
        }
        this.leaveQueue(sim.serverThreads);
        this.leaveQueue(sim.server);
    }
}

let cacheRatio = ref(0.9);
let cacheLatency = ref(2);  // ms
let cacheSTDDev = ref(0.5); // ms
let dbLatency = ref(20);  // ms
let dbSTDDev = ref(5);  // ms
let arrivalRate=ref(200);  // per sec
const data = ref(null);
const running = ref(false);

function start() {
    const sim = new CacheAndDB(20, arrivalRate.value, cacheRatio.value, cacheLatency.value, cacheSTDDev.value, dbLatency.value, dbSTDDev.value);
    console.log("Running simulation");
    sim.start();
    running.value = true;

    sim.finished.addEventListener(() => {
        data.value = {
            timeNow: sim.timeNow,
            wallClock: sim.timeElapsed,
            meanUsedThreads: sim.serverThreads.capacity,
            populationTable: sim.getStatsTable(),
            serverHistogram: sim.server.grossDwell.getHistogramChart("Overall latency (ms)"),
            dbHistogram: sim.qDB.grossDwell.getHistogramChart("DB latency (ms)"),
            cacheHistogram: sim.qCache.grossDwell.getHistogramChart("Cache latency (ms)")
        };
        running.value = false;        
    });

}

</script>