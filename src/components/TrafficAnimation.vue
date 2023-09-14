<template>
    <div>
        <label for="speed-slider">Adjust Time Speed:</label>
        <input type="range" v-model="speed" min="1" max="10" />
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
            width: 600,
            height: 600,
            trafficRoad1: null,
            trafficRoad2: null,
            eastSideQueue1: null, //Car in the east-side is moving to west, so the queue in the eastSide is westMoving cars
            eastSideQueue2: null,
            westSideQueue1: null,
            westSideQueue2: null,
            block: null,
            car1: null,
            car2: null,
            timer: null,
            blockWidth: 50,

            roadWidth: 300,
            roadHeight: 27,
            roadStart: 350,
            roadEnd: 520,
            carRadius: 10,
            eastMovingData: null,
            queueColor1: d3.rgb(172, 250, 198),
            queueColor2: d3.color("#17bd86ec"),
            roadColor: d3.rgb(170, 238, 207),
            road2Color: d3.color("#069e6bec"),
            speed: 5,
            simulationSpeedReduction: 100, //1 second = 1000, so this speed is used to adjust the simulation speed, displaySpeed = simulationTime *1000 / simulationSpeedReduction ;

            carData: [],
            carQueue: [],
            timeSpeed: 5,
            currentTime: 0,
            objectiveTime: 0,
            lastRender: null,
            carsInQueue: 0
        }
    },
    mounted() {

        this.carData = sorted_data.map(d => ({ ...d, processed: false }));
        this.svg = d3.select("svg");
        this.width = +this.svg.attr("width");
        this.height = +this.svg.attr("height");
        this.roadEnd = this.roadStart + this.roadWidth;

        this.westSideQueue1 = this.svg.append("rect")
            .attr("x", 0)
            .attr("y", this.height / 2 - this.roadHeight)
            .attr("width", this.roadStart)
            .attr("height", this.roadHeight)
            .attr("class", "road")
            .attr("fill", this.queueColor1);

        this.westSideQueue2 = this.svg.append("rect")
            .attr("x", 0)
            .attr("y", this.height / 2)
            .attr("width", this.roadStart)
            .attr("height", this.roadHeight)
            .attr("class", "road")
            .attr("fill", this.queueColor2);


        this.eastSideQueue1 = this.svg.append("rect")
            .attr("x", this.roadEnd)
            .attr("y", this.height / 2 - this.roadHeight)
            .attr("width", this.roadStart)
            .attr("height", this.roadHeight)
            .attr("class", "road")
            .attr("fill", this.queueColor1);

        this.eastSideQueue2 = this.svg.append("rect")
            .attr("x", this.roadEnd)
            .attr("y", this.height / 2)
            .attr("width", this.roadStart)
            .attr("height", this.roadHeight)
            .attr("class", "road")
            .attr("fill", this.queueColor2);

        this.trafficRoad1 = this.svg.append("rect")
            .attr("x", this.roadStart)
            .attr("y", this.height / 2 - this.roadHeight)
            .attr("width", this.roadWidth)
            .attr("height", this.roadHeight)
            .attr("class", "road")
            .attr("fill", this.roadColor);

        this.trafficRoad2 = this.svg.append("rect")
            .attr("x", this.roadStart)
            .attr("y", this.height / 2)
            .attr("width", this.roadWidth)
            .attr("height", this.roadHeight)
            .attr("class", "road")
            .attr("fill", this.road2Color);

        this.block = this.svg.append("rect")
            .attr("x", this.roadStart + (this.roadWidth / 2) - (this.blockWidth / 2)) // Adjust block to the center of trafficRoad1
            .attr("y", this.height / 2 - this.roadHeight)
            .attr("width", this.blockWidth)
            .attr("height", this.roadHeight)
            .attr("class", "block");

        // Adjust car1 initial position to align with the middle of trafficRoad1
        this.car1 = this.svg.append("circle")
            .attr("r", this.carRadius)
            .attr("cx", this.roadStart + this.carRadius)
            .attr("cy", this.height / 2 - this.roadHeight / 2)
            .attr("fill", "red");

        this.car2 = this.svg.append("circle")
            .attr("r", this.carRadius)
            .attr("cx", this.roadEnd - this.carRadius)
            .attr("cy", this.height / 2 + this.roadHeight / 2)
            .attr("fill", "blue");

        this.animate();
    },
    beforeUnmount() {
        cancelAnimationFrame(this.animationFrame);
    },
    watch: {
        speed() {
            // Interrupt ongoing animations and restart the animation with the new speed
            this.car1.interrupt();
            this.car2.interrupt();

            this.animate();
        }
    },
    methods: {

        animate() {
            var speed = 10000 / this.speed;
            this.animateCar1(speed);
            this.animateCar2(speed);
            this.animationQueueInWest();
            this.monitorQueueAndMoveCars();
        },



        animationQueueInWest(timestamp = 0) {
            if (this.lastRender === null) {
                this.lastRender = timestamp;
                this.animationFrame = requestAnimationFrame(this.animationQueueInWest);
                return;
            }
            const delta = (timestamp - this.lastRender) / 1000;
            this.lastRender = timestamp;

            this.objectiveTime += delta * this.timeSpeed;
            this.carData.forEach((d, index) => { // Add index to the forEach callback
                if (!d.processed && this.objectiveTime >= d.eventArrivingTime) {
                    d.processed = true;
                    this.currentTime = parseFloat(d.eventArrivingTime);
                    this.carsInQueue++;

                    const car = this.svg.append("circle")
                        .attr("cx", 0)
                        .attr("cy", this.height / 2 - this.roadHeight / 2)
                        .attr("r", this.carRadius)
                        .attr("fill", "red");

                    const finalPosition = this.roadStart - (this.carsInQueue * 2 * this.carRadius) + this.carRadius;
                    console.log(this.carsInQueue);
                    console.log("finalPosition" + finalPosition)

                    car.transition()
                        .duration((finalPosition - 0) / this.timeSpeed * 10)
                        .attr("cx", finalPosition);

                    // Add the car to the queue with its index
                    this.carQueue.push({ car: car.node(), index });
                }
            });

            this.animationFrame = requestAnimationFrame(this.animationQueueInWest);
        },

        monitorQueueAndMoveCars() {
            console.log("queue length" + this.carQueue.length);
            for (let index = 0; index < this.carQueue.length; index++) {
                const carObject = this.carQueue[index];
                const car = d3.select(carObject.car);
                const carData = this.carData[carObject.index];
                const currentTime = this.objectiveTime;
                const passingStartTime = parseFloat(carData.eventArrivingTime) + parseFloat(carData.waitingTime);
                let self = this;
                if (currentTime >= passingStartTime) {
                    car.transition()
                        .duration(1000)
                        .ease(d3.easeLinear)
                        .attr("cx", self.roadStart + self.carRadius)
                        .on('end', () => {
                            car.remove();
                        });
                    this.carQueue.splice(index, 1); // Remove the car from the queue
                    index--; // Adjust the index to account for the removed car
                    this.carsInQueue--;
                }

            }

            requestAnimationFrame(this.monitorQueueAndMoveCars);
        },



        animateCar1(speed) {
            this.car1.transition()
                .duration(speed)
                .ease(d3.easeLinear)
                .attr("cx", this.roadEnd - this.carRadius)
                .tween("pathTween", () => {
                    var car = d3.select(this.car1.node());
                    var blockD3 = d3.select(this.block.node());

                    return () => {
                        var currentX = parseFloat(car.attr("cx"));
                        if (Math.abs(currentX - parseFloat(blockD3.attr("x"))) < 20 && car.attr("cy") == (this.height / 2 - this.roadHeight / 2)) {
                            car.attr("cy", this.height / 2 + this.roadHeight / 2);
                        }

                        if (currentX > parseFloat(blockD3.attr("x")) + parseFloat(blockD3.attr("width")) && car.attr("cy") == (this.height / 2 + this.roadHeight / 2)) {
                            car.interrupt().attr("cy", this.height / 2 - this.roadHeight / 2).attr("cx", parseFloat(blockD3.attr("x")) + parseFloat(blockD3.attr("width")) + this.carRadius);
                            var remainingDistance = this.roadEnd - parseFloat(car.attr("cx"));
                            var remainingTime = (speed * remainingDistance) / this.roadWidth;
                            car.transition()
                                .duration(remainingTime)
                                .ease(d3.easeLinear)
                                .attr("cx", this.roadEnd - this.carRadius)
                                .on("end", () => {
                                    car.attr("cy", this.height / 2 - this.roadHeight / 2)
                                        .transition()
                                        .duration(speed)
                                        .ease(d3.easeLinear)
                                        .attr("cx", this.roadStart + this.carRadius)
                                        .on("end", this.animate);
                                });
                        }
                    };
                })
                .on("end", () => {
                    d3.select(this.car1.node())
                        .attr("cy", this.height / 2 - this.roadHeight / 2)
                        .transition()
                        .duration(speed)
                        .ease(d3.easeLinear)
                        .attr("cx", this.roadStart + this.carRadius)
                        .on("end", this.animate);
                });
        },
        animateCar2(speed) {
            this.car2.transition()
                .duration(speed)
                .ease(d3.easeLinear)
                .attr("cx", this.roadStart + this.carRadius)
                .on("end", () => {
                    d3.select(this.car2.node())
                        .transition()
                        .duration(speed)
                        .ease(d3.easeLinear)
                        .attr("cx", this.roadEnd - this.carRadius)
                        .on("end", this.animate);
                });
        }
    }
}
</script>

<style scoped>
.road {
    fill: rgb(170, 238, 207);
    stroke: rgb(29, 1, 1);
    stroke-width: 2;
}

.block {
    fill: #17bd86ec;
}
</style>
