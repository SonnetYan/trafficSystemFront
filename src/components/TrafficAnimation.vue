<template>
    <div>
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
            eastSideQueue:null,
            westSideQueue:null,
            block: null,
            car1: null,
            car2: null,
            blockWidth: 50,
            trafficRoadWidth:600,
            roadStart: 20, // Road start position
            roadEnd: 620, // Road end position

            roadColor: d3.rgb(170, 238, 207),
            road2Color:d3.color("#069e6bec")
        }
    },
    mounted() {
        this.svg = d3.select("svg");
        this.width = +this.svg.attr("width");
        this.height = +this.svg.attr("height");

        this.trafficRoad1 = this.svg.append("rect")
            .attr("x", this.roadStart)
            .attr("y", this.height / 2 - 10)
            .attr("width", this.trafficRoadWidth)
            .attr("height", 20)
            .attr("class", "road")
            .attr("fill", this.roadColor);

        this.trafficRoad2 = this.svg.append("rect")
            .attr("x", this.roadStart)
            .attr("y", this.height / 2 + 10)
            .attr("width", this.trafficRoadWidth)
            .attr("height", 20)
            .attr("class", "road")
            .attr("fill", this.road2Color);

        this.block = this.svg.append("rect")
            .attr("x", (this.trafficRoadWidth - this.blockWidth) / 2)
            .attr("y", this.height / 2 - 10)
            .attr("width", this.blockWidth)
            .attr("height", 20)
            .attr("class", "block");

        this.car1 = this.svg.append("circle")
            .attr("r", 10)
            .attr("cx", this.roadStart)
            .attr("cy", this.height / 2 - 0)
            .attr("fill", "red");

        this.car2 = this.svg.append("circle")
            .attr("r", 10)
            .attr("cx", this.roadEnd)
            .attr("cy", this.trafficRoadWidth / 2 + 20)
            .attr("fill", "blue");

        this.animate();
    },
    methods: {
      animate() {
    this.car1.transition()
        .duration(5000)
        .ease(d3.easeLinear)
        .attr("cx", this.roadEnd);

    this.car2.transition()
        .duration(5000)
        .ease(d3.easeLinear)
        .attr("cx", this.roadStart);
}

    }
}
</script>

<style scoped>
.road {
    fill: rgb(170, 238, 207);
    stroke:rgb(29, 1, 1);
    stroke-width: 2;
}

.block {
    fill: #069e6bec;
}
</style>




