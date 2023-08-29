<template>
    <div>
        <input type="range" v-model="speed" min="1" max="10" />
        <svg width="1000" height="600"></svg>
    </div>
</template>

<script>
import * as d3 from 'd3';

export default {
    name: 'TrafficAnimation',
    data() {
        return {
            svg: null,
            width: 600,
            height: 600,
            trafficRoad1: null,
            trafficRoad2: null,
            eastSideQueue1 : null, //Car in the east-side is moving to west, so the queue in the eastSide is westMoving cars
            eastSideQueue2 : null,
            westSideQueue1: null,
            westSideQueue2: null,
            block: null,
            car1: null,
            car2: null,
            blockWidth: 50,

            roadWidth: 300,
            roadHeight: 27,
            roadStart: 350,
            roadEnd: 520,
            carRadius: 10,

            queueColor1: d3.rgb(172,250,198),
            queueColor2 : d3.color("#17bd86ec"),
            roadColor: d3.rgb(170, 238, 207),
            road2Color: d3.color("#069e6bec"),
            speed: 5
        }
    },
    mounted() {
        this.svg = d3.select("svg");
        this.width = +this.svg.attr("width");
        this.height = +this.svg.attr("height");
        this.roadEnd = this.roadStart + this.roadWidth;

        this.westSideQueue1 = this.svg.append("rect")
            .attr("x", 0)
            .attr("y", this.height / 2 - this.roadHeight)
            .attr("width",this.roadStart )
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


            this.westSideQueue1 = this.svg.append("rect")
            .attr("x", this.roadEnd)
            .attr("y", this.height / 2 - this.roadHeight)
            .attr("width",this.roadStart )
            .attr("height", this.roadHeight)
            .attr("class", "road")
            .attr("fill", this.queueColor1);

        this.westSideQueue2 = this.svg.append("rect")
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
            .attr("x", this.roadStart + (this.roadWidth  / 2) - (this.blockWidth / 2)) // Adjust block to the center of trafficRoad1
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
            var speed  = 10000 / this.speed;
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
