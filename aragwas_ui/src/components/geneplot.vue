<template>
    <div>
        <svg id="chart" :height="height" :width="width" ref="svg">
            <g class="manhattanplot">
                <g class="x axis" :transform="'translate(0,'+(paddedScatter.height)+')'">
                    <text style="text-anchor:middle;" fill="black" :transform="'translate('+ width / 2+', 30)'">chromosomal position [{{options.chr }}]</text>
                </g>
                <g class="y axis" :transform="'translate('+margin.left+',0)'">
                    <text style="text-anchor:middle;" fill="black" :transform="'translate(-25,'+paddedScatter.height /2 +') rotate(-90)'">-log10(p-value)</text>
                </g>
                <g class="fdr-legend" transform="translate(5, 0)">
                    <rect fill="rgb(0,100,0)" height="10" width="16" :x="margin.left" y="3"></rect>
                    <text style="text-anchor:middel" fill="black" :transform="'translate('+(margin.left + 24)+',12)'" >Bonferroni threshold [{{alpha}}]</text>
                </g>
                <line id="fdr-bonferroni" stroke="rgb(0,100,0)" stroke-width="1.5"></line>
                </g>
            <g class="geneplot" :transform="'translate('+ margin.left + ','+ (scatterPlotHeight ) +')'">

            </g>
        </svg>
    </div>
</template>

<script lang="ts">
    import * as d3 from "d3";
    import Vue from "vue";
    import {Component, Prop, Watch} from "vue-property-decorator";

    import Gene, {GenePlotOptions} from "../models/gene";

    @Component({
        name: "gene-plot",
    })
    export default class GenePlot extends Vue {
        @Prop()
        dataPoints;

        @Prop()
        genes: Gene[];

        @Prop()
        options: GenePlotOptions;

        width: number = 0;
        scatterPlotHeight: number = 300;
        genePlotHeight: number = 250;
        readonly alpha: number = 0.05;
        readonly padding = 40;
        scales = {x: d3.scaleLinear(), y: d3.scaleLinear()};
        axis = {x: d3.axisBottom(this.scales.x), y: d3.axisLeft(this.scales.y)};

        readonly margin = {
            left: 40,
            right: 10,
            bottom: 40,
            top: 40,
        }

        get height() {
            return this.scatterPlotHeight + this.genePlotHeight;
        }

        get paddedScatter() {
            const width = this.width - this.margin.left - this.margin.right;
            const height = this.scatterPlotHeight - this.margin.top - this.margin.bottom;
            return { width, height };
        }

        @Watch("width")
        onWidthChanged(newWidth: number, oldWidth: number) {
            this.draw();
        }

        @Watch("options")
        onOptionsChanged(newOptions: GenePlotOptions, oldOptions: GenePlotOptions) {
            this.draw();
        }

        @Watch("genes")
        onGenesChanged() {
            this.drawGenePlot();
        }

        mounted() {
            window.addEventListener('resize', this.onResize);
            this.onResize();
        }

        beforeDestroy() {
            window.removeEventListener('resize', this.onResize);
        }

        onResize() {
            this.width = this.$el.offsetWidth;
            this.scales.x.range([this.margin.left, this.paddedScatter.width]);
            this.scales.y.range([this.paddedScatter.height, this.margin.top]);
            this.axis.x.scale(this.scales.x);
            this.axis.y.scale(this.scales.y);
            this.draw();
        }

        drawManhattan() {
            this.scales.x.domain([this.options.startPos, this.options.endPos])
            this.scales.y.domain([0, this.options.maxScore + 1])
            const svg = d3.select(this.$refs.svg as Element)
            svg.select("g.y.axis")
                .call(this.axis.y);
            svg.select("g.x.axis")
                .call(this.axis.x);

            svg.select("line#fdr-bonferroni")
                .attr("x1", this.margin.left)
                .attr("x2", this.paddedScatter.width)
                .attr("y1",this.scales.y(this.options.bonferoniThreshold))
                .attr("y2",this.scales.y(this.options.bonferoniThreshold)) ;


            /*svg.selectAll("circle")
                .data(this.dataPoints)
                .enter()
                .append("circle")
                .attr("cx", (d) => {
                    return scaleW(d[0]);
                })
                .attr("cy", (d) => {
                    return scaleH(d[1]);
                })
                .attr("r", 2.1)
                .style("fill", "rgb(" + red + ",102," + blue + ")");*/
        }

        drawGenePlot() {
            const svg = d3.select(this.$refs.svg as Element)
            var geneNodes = svg.select("g.geneplot")
                .selectAll("rect.genes")
                .data(this.genes);

            // remove old ones
            geneNodes.exit().remove();

            geneNodes.enter().append("rect") // ENTER
                .attr("class","genes")
                .style("fill", "green")
            .merge(geneNodes)
                .attr("x",(d) => this.scales.x(d.positions.gte))
                .attr("y",10)
                .attr("width", (d) => this.scales.x(d.positions.lte) - this.scales.x(d.positions.gte) )
                .attr("height", 20)
                .style("fill", "rgb(255,255,255)")
                .style("stroke", "rgb(0,100,0)")
                .style("stroke-width", 1);
        }

        draw() {
            this.drawManhattan();
            this.drawGenePlot();
        }
    }
</script>
<style scoped>

</style>
