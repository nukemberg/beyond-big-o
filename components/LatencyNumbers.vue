<template>
    <div class="container rows-2">
        <div id="slider mb-10">
            <span>Year: {{ year }}</span>
            <input class="mx-4" type="range" name="year" id="slider-year" min="1990" max="2024" v-model="year">
        </div>
        <div class="grid grid-cols-4">
            <template v-for="(obj, key) in data">
                <div class="">
                    <template v-for="(item, itemName) in obj">
                        <div class="grid grid-cols-2 object my-2">
                            <div>
                                <svg :id="key + '-' + itemName">
                                </svg>
                            </div>
                            <div class="">{{ item.description }}
                                <svg v-if="item.descriptionElementStyle" width="11" height="11" style="display: inline;">
                                    <rect height="10" width="10" x="0" y="0" :style="item.descriptionElementStyle"></rect>
                                </svg>
                                <span v-else>{{ item.value.toLocaleString("en", { maximumFractionDigits: 0, style: "unit", unit: "nanosecond", unitDisplay: "narrow"}) }} ≈ {{ math.unit(item.value, "ns").format({notation: "fixed", precision: 0}) }}</span>
                            </div>
                        </div>
                    </template>
                </div>
            </template>
        </div>
    </div><!--/.fluid-container-->
</template>

<style scoped>
.object {
    font-size: small;
}
</style>

<script setup>
import {computed, defineModel, onUpdated, onMounted} from "vue";
import * as d3 from "d3";
import * as math from "mathjs";

const year = defineModel("year", {default: new Date().getFullYear()});

function shift(_year) {
    return _year - 1982;
}

function getPayloadBytes() {
    // 1 MB
    return Math.pow(10, 6);
}
function getNetworkPayloadBytes() {
    // 2KB
    return 2 * Math.pow(10, 3);
}
function getCycle() {
    // Clock speed stopped at ~3GHz in ~2005
    // [source: http://www.kmeme.com/2010/09/clock-speed-wall.html]
    // Before then, doubling every ~2 years
    // [source: www.cs.berkeley.edu/~pattrsn/talks/sigmod98-keynote.ppt]
    if (year.value <= 2005) {
        // 3*10^9 = a*b^(2005-1990)
        // b = 2^(1/2)
        // -> a = 3*10^9 / 2^(2005.5)
        var a = 3 * Math.pow(10, 9) / Math.pow(2, shift(2005) * 0.5);
        var b = Math.pow(2, 1.0 / 2);
        var hz = a * Math.pow(b, shift(year.value));
    } else {
        var hz = 3 * Math.pow(10, 9);
    }
    //  1 / HZ = seconds
    //  1*10^9 / HZ = ns
    var ns = Math.pow(10, 9) / hz;
    return ns;
}
function getMemLatency() {
    // Bus Latency is actually getting worse:
    // [source: http://download.micron.com/pdf/presentations/events/winhec_klein.pdf]
    // 15 years ago, it was decreasing 7% / year
    // [source: www.cs.berkeley.edu/~pattrsn/talks/sigmod98-keynote.ppt]
    if (year.value <= 2000) {
        // b = 0.93
        // 100ns = a*0.93^2000
        /// -> a = 100 / 0.93^2000
        var b = 0.93;
        var a = 100.0 / Math.pow(0.93, shift(2000));
        var ms = a * Math.pow(b, shift(year.value));
    } else {
        var ms = 100; // ns
    }
    return ms;
}
function getNICTransmissionDelay(payloadBytes) {
    // NIC bandwidth doubles every 2 years
    // [source: http://ampcamp.berkeley.edu/wp-content/uploads/2012/06/Ion-stoica-amp-camp-21012-warehouse-scale-computing-intro-final.pdf]
    // TODO: should really be a step function
    // 1Gb/s = 125MB/s = 125*10^6 B/s in 2003
    // 125*10^6 = a*b^x
    // b = 2^(1/2)
    // -> a = 125*10^6 / 2^(2003.5)
    var a = 125 * Math.pow(10, 6) / Math.pow(2, shift(2003) * 0.5);
    var b = Math.pow(2, 1.0 / 2);
    var bw = a * Math.pow(b, shift(year.value));
    // B/s * s/ns = B/ns
    var ns = payloadBytes / (bw / Math.pow(10, 9));
    return ns;
}
function getBusTransmissionDelay(payloadBytes) {
    // DRAM bandwidth doubles every 3 years
    // [source: http://ampcamp.berkeley.edu/wp-content/uploads/2012/06/Ion-stoica-amp-camp-21012-warehouse-scale-computing-intro-final.pdf]
    // 1MB / 250,000 ns = 10^6B / 0.00025 = 4*10^9 B/s in 2001
    // 4*10^9 = a*b^x
    // b = 2^(1/3)
    // -> a = 4*10^9 / 2^(2001.33)
    var a = 4 * Math.pow(10, 9) / Math.pow(2, shift(2001) * 0.33);
    var b = Math.pow(2, 1.0 / 3);
    var bw = a * Math.pow(b, shift(year.value));
    // B/s * s/ns = B/ns
    var ns = payloadBytes / (bw / Math.pow(10, 9));
    return ns;
}
function getSSDLatency() {
    // Will flat-line in one capacity doubling cycle (18 months=1.5years)
    // Before that, 20X decrease in 3 doubling cycles (54 months=4.5years)
    // Source: figure 4 of http://cseweb.ucsd.edu/users/swanson/papers/FAST2012BleakFlash.pdf
    // 20 us = 2*10^4 ns in 2012
    // b = 1/20 ^ 1/4.5
    // -> a = 2*10^4 / 1/20 ^(2012 * 0.22)
    if (year.value <= 2014) {
        var a = 2 * Math.pow(10, 4) / Math.pow(1.0 / 20, shift(year.value) * 0.22);
        var b = Math.pow(1.0 / 20, 1.0 / 4.5);
        return a * Math.pow(b, shift(year.value));
    } else {
        return 16000;
    }
}
function getSSDTransmissionDelay(payloadBytes) {
    // SSD bandwidth doubles every 3 years
    // [source: http://ampcamp.berkeley.edu/wp-content/uploads/2012/06/Ion-stoica-amp-camp-21012-warehouse-scale-computing-intro-final.pdf]
    // 3GB/s = a*b^2012
    // b = 2^(1/3)
    // -> a = 6*10^9 / 2^(2012.33)
    let a = 3 * Math.pow(10, 9) / Math.pow(2, shift(2012) * 0.33);
    let b = Math.pow(2, 1.0 / 3);
    let bw = a * Math.pow(b, shift(year.value));
    // B/s * s/ns = B/ns
    let ns = payloadBytes / (bw / Math.pow(10, 9));
    return ns;
}
function getSeek() {
    // Seek + rotational delay halves every 10 years
    // [source: http://www.storagenewsletter.com/news/disk/hdd-technology-trends-ibm]
    // In 2000, seek + rotational =~ 10 ms
    // b = (1/2)^(1/10)
    // -> a = 10^7 / (1/2)^(2000*0.1)
    var a = Math.pow(10, 7) / Math.pow(0.5, shift(2000) * 0.1);
    var b = Math.pow(0.5, 0.1);
    var ns = a * Math.pow(b, shift(year.value));
    return ns;
}
function getDiskTransmissionDelay(payloadBytes) {
    // Disk bandwidth is increasing very slowly -- doubles every ~5 years
    // [source: http://ampcamp.berkeley.edu/wp-content/uploads/2012/06/Ion-stoica-amp-camp-21012-warehouse-scale-computing-intro-final.pdf]
    // Before 2002 (~100MB/s):
    // Disk bandwidth doubled every two years
    // [source: www.cs.berkeley.edu/~pattrsn/talks/sigmod98-keynote.ppt]
    if (year.value <= 2002) {
        // 100MB/s = a*b^2002
        // b = 2^(1/2)
        // -> a = 10^8 / 2^(2002.5)
        var a = Math.pow(10, 8) / Math.pow(2, shift(2002) * 0.5);
        var b = Math.pow(2, 1.0 / 2);
        var bw = a * Math.pow(b, shift(year.value));
    } else {
        // 100MB/s = a*b^2002
        // b = 2^(1/5)
        // -> a = 10^8 / 2^(2002-1982 * .2)
        var a = Math.pow(10, 8) / Math.pow(2, shift(2002) * 0.2);
        var b = Math.pow(2, 1.0 / 5);
        var bw = a * Math.pow(b, shift(year.value));
    }
    // B/s * s/ns = B/ns
    var ns = payloadBytes / (bw / Math.pow(10, 9));
    return ns;
}
function getDCRTT() {
    // Assume this doesn't change much?
    return 500000; // ns
}
function getWanRTT() {
    // Routes are arguably improving:
    //   http://research.google.com/pubs/pub35590.html
    // Speed of light is ultimately fundamental
    return 150000000; // ns
}

function colorForUnit(unit) {
    switch(unit) {
        case "1ns":
            return "rgb(220, 220, 220)";
        case "100ns":
            return "rgb(40, 0, 220)";
        case "10us":
            return "green";
        case "1ms":
            return "rgb(255, 0, 0)";
    }
}

function diagram(el, n, unit) {
    let boxes = math.unit(n, "ns").divide(math.unit(unit));
    let color = colorForUnit(unit);
    let length = 10;
    let cw = 100;
    let ch = Math.ceil(boxes / 10)*length;

    let data = Array(Math.floor(boxes)).fill(1);
    const remainder = boxes - Math.floor(boxes);
    if (remainder > 0) {
        data.push(remainder);
    }
    

    let rects = d3.select(el).
        attr("width", cw).
        attr("height", ch)
        .selectAll("rect")
        .data(data)
        .join("rect")
        .attr("x", (d, idx) => length*(idx%10))
        .attr("y", (d, idx) => Math.floor(idx/10)*length)
        .attr("height", length)
        .attr("width", (d) => d*length)
        .attr("fill", color);

    return rects;
}

let black = "stroke:#FFFFFF; fill: #000000";
let blue = "stroke:#FFFFFF; fill: #0000FF";
let green = "stroke:#FFFFFF; fill: #00CC00";
let red = "stroke:#FFFFFF; fill: #FF0000";

function addCommas(nStr) {
    // First, make fixed length
    // TODO: bad separation of concerns
    if (nStr < 1) {
        nStr = nStr.toFixed(1);
    } else {
        nStr = nStr.toFixed(0);
    }
    nStr += '';
    let x = nStr.split('.');
    let x1 = x[0];
    let x2 = x.length > 1 ? '.' + x[1] : '';
    let rgx = /(\d+)(\d{3})/;
    while (rgx.test(x1)) {
        x1 = x1.replace(rgx, '$1' + ',' + '$2');
    }
    return x1 + x2;
}

const data = computed(() => ({
    cpu: {
        L1: {value: 3*getCycle(), unit: "1ns", description: "L1 cache reference", suffix: "L1"},
        branch: {value: 10*getCycle(), unit: "1ns", description: "Branch mispredict"},
        L2: {value: 13 * getCycle(), unit: "1ns", description: "L2 cache reference"},
        mutex: {value: 50 * getCycle(), unit: "1ns", description: "Mutex lock/unlock"},
        ns100: {value: 100, unit: "1ns",  description: "100ns = ", descriptionElementStyle: blue}
    },
    memory: {
        mem: {value: getMemLatency(), unitN: 100, unit: "100ns", description: "Main memory reference"},
        micro: {value: 100 * 10, unit: "100ns", description: "10,000ns ≈ 10μs"},
        zippy: {value: 6000 * getCycle(), unit: "100ns", description: "Compress 1KB wth Zippy"},
        tenMicro: {value: 100 * 100, unit: "100ns", description: "10μs = ", descriptionElementStyle: green}
    },
    network: {
        network: {value: getNICTransmissionDelay(getNetworkPayloadBytes()), unit: "10us", description: `Send ${addCommas(getNetworkPayloadBytes())} bytes over commodity network`},
        ssdRandom: {value: getSSDLatency(), unit: "10us", description: "SSD random read"},
        mbMem: {value: getBusTransmissionDelay(getPayloadBytes()), unit: "10us", description: `Read ${addCommas(getPayloadBytes())} bytes sequentially from memory`},
        rtt: {value: getDCRTT(), unit: "10us", description: "Round trip in same datacenter"},
        ms: {value: 100 * 100 * 100, unit: "10us", description: "10ms = ", descriptionElementStyle: red}
    },
    disk: {
        mbSSD: {value: getSSDTransmissionDelay(getPayloadBytes()), unit: "1ms", description: `Read ${addCommas(getPayloadBytes())} bytes sequentially from SSD`},
        seek: {value: getSeek(), unit: "1ms", description: "Disk seek"},
        mbDisk: {value: getDiskTransmissionDelay(getPayloadBytes()), unit: "1ms", description: `Read ${addCommas(getPayloadBytes())} bytes sequentially from disk`},
        wan: {value: getWanRTT(), unit: "1ms", description: "Packet roundtrip CA to Netherlands"}
    }
}));

function d3Draw() {
    Object.entries(data.value).forEach(([k, obj]) => {
        Object.entries(obj).forEach(([subkey, item]) => {
            diagram(`#${k}-${subkey}`, item.value, item.unit);
        }) 
    });
}

onMounted(d3Draw);
onUpdated(d3Draw);
</script>
