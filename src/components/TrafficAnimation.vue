<template>
    <div>
        <label for="speed-slider">Adjust Time Speed:</label>
        <input id="speed-slider" type="range" v-model="timeSpeed" min="1" max="10" />
        <svg width="1000" height="600"></svg>
        <div class="timer">Current Time: {{ typeof currentTime === 'number' ? currentTime.toFixed(2) : '0.00' }}</div>
        <div class="objective-timer">Objective Time: {{ isNaN(objectiveTime) ? '0.00' : objectiveTime.toFixed(2) }}</div>
    </div>
</template>

<script>
import * as d3 from 'd3';
import sorted_data from '../data/sorted_data.json';

export default {
    name: 'TrafficAnimation',
    data() {
        return {
            svg: null,
            width: 1000,
            height: 600,
            carRadius: 10,
            carData: [],
            queueStart: 200, 
            queueEnd: 800,
            currentTime: 0,
            objectiveTime: 0,
            timeSpeed: 5,
            lastRender: null,
            carsInQueue: 0
        }
    },
    mounted() {
        this.carData = sorted_data.map(d => ({...d, processed: false}));
        this.svg = d3.select("svg");
        this.width = +this.svg.attr("width");
        this.height = +this.svg.attr("height");

        this.svg.append("rect")
            .attr("x", this.queueStart)
            .attr("y", this.height / 2 - this.carRadius)
            .attr("width", this.queueEnd - this.queueStart)
            .attr("height", this.carRadius * 2)
            .attr("fill", "#e0e0e0")
            .attr("stroke", "black");

        this.animate();
    },
    beforeUnmount() {
        cancelAnimationFrame(this.animationFrame);
    },
    methods: {
        animate(timestamp = 0) {
            if (this.lastRender === null) {
                this.lastRender = timestamp;
                this.animationFrame = requestAnimationFrame(this.animate);
                return;
            }
            
            const delta = (timestamp - this.lastRender) / 1000;  
            this.lastRender = timestamp;

            this.objectiveTime += delta * this.timeSpeed;

            this.carData.forEach((d) => {
                if (!d.processed && this.objectiveTime >= d.eventArrivingTime) {
                    d.processed = true;
                    this.currentTime = parseFloat(d.eventArrivingTime);
                    this.carsInQueue++;

                    const car = this.svg.append("circle")
                        .attr("cx", this.queueStart)
                        .attr("cy", this.height / 2)
                        .attr("r", this.carRadius)
                        .attr("fill", "blue");
                    
                    const finalPosition = this.queueEnd - (this.carsInQueue * 2 * this.carRadius)+ this.carRadius;

                    car.transition()
                        .duration((finalPosition - this.queueStart) / this.timeSpeed * 10)
                        .attr("cx", finalPosition);
                }
            });

            this.animationFrame = requestAnimationFrame(this.animate);
        }
    }
}
</script>

<style scoped>
.timer, .objective-timer {
    font-size: 20px;
    margin-top: 10px;
}
</style>
