<template>
  <svg id="context" />
</template>

<script>
import * as d3 from "d3";

export default {
  name: "ForcedGraph",
  data: () => ({
    nodes: [],
    links: [],

    // mouse coordinates we save on each mouse move.
    mouse: null,

    // for creating link between the nodes
    isCreatingLink: false,
    nodeFrom: null,
    nodeTo: null,
    mouselink: null,

    //D3 elements
    svg: null,
    linksElem: null,
    nodesElem: null,

    simulation: null
  }),
  props: {
    // viewport width
    width: {
      type: Number,
      default: 500
    },
    //viewport height
    height: {
      type: Number,
      default: 500
    },
    nodeRadius: {
      type: Number,
      default: 5
    },
    nodeColor: {
      type: String,
      default: "black"
    },
    nodeHoveredColor: {
      type: String,
      default: "orange"
    },
    linkWidth: {
      type: Number,
      default: 1.5
    },
    linkColor: {
      type: String,
      default: "red"
    }
  },
  methods: {
    init: function() {
      this.svg = d3
        .select("#context")
        .attr("viewBox", [
          -this.width / 2,
          -this.height / 2,
          this.width,
          this.height
        ])
        .on("mousedown", this.onSvgMouseDown)
        .on("mousemove", this.onSvgMouseMove)
        .on("mouseup", this.onSvgMouseUp);

      // links
      this.linksElem = this.svg.append("g").selectAll("line");

      // nodes
      this.nodesElem = this.svg.append("g").selectAll("circle");

      // temp link line
      this.mouselink = this.svg
        .append("g")
        .attr("stroke", "red")
        .selectAll("line");

      this.simulation = d3
        .forceSimulation(this.nodes)
        .force(
          "link",
          d3.forceLink(this.links).id(d => d.index)
        )
        .force("charge", d3.forceManyBody().strength(-60))
        .force("x", d3.forceX())
        .force("y", d3.forceY())
        .on("tick", this.ticked);
    },

    onNodeMouseDown: function(d, i, svgNodes) {
      d3.event.stopPropagation();
      if (this.isCreatingLink === false) {
        this.isCreatingLink = true;
        this.nodeFrom = this.nodes[i];
      }
    },

    onNodeMouseUp: function(d, i, svgNodes) {
      d3.event.stopPropagation();
      if (this.isCreatingLink === true) {
        this.nodeTo = this.nodes[i];
        if (this.nodeTo == null) {
          return;
        }

        if (this.nodeTo.index === this.nodeFrom.index) {
          this.nodeFrom = null;
          this.isCreatingLink = false;
          this.nodeTo = null;
          this.nodes.splice(i, 1);

          this.links = this.links.filter(function(l) {
            return l.source !== d && l.target !== d;
          });
          this.restartSimulation();
          return;
        }

        this.links = [
          ...this.links,
          { source: this.nodeFrom, target: this.nodeTo }
        ];
        this.restartSimulation();

        this.isCreatingLink = false;
        this.nodeFrom = null;
        this.nodeTo = null;
      }
    },

    onNodeMouseOver: function(d, i, svgNodes) {
      d3.select(svgNodes[i]).attr("fill", this.nodeHoveredColor);
    },

    onNodeMouseOut: function(d, i, svgNodes) {
      d3.select(svgNodes[i]).attr("fill", this.nodeColor);
    },

    onSvgMouseMove: function(d, i, svgNodes) {
      const [x, y] = d3.mouse(svgNodes[i]);
      this.mouse = { x, y };
    },

    onSvgMouseUp: function(d, i, svgNodes) {
      console.log("svg mouseup");
      if (this.isCreatingLink) {
        this.isCreatingLink = false;
        this.nodeFrom = null;
        this.nodeTo = null;
      }
    },

    onSvgMouseDown: function(d, i, svgNodes) {
      d3.event.stopPropagation();
      const node = { x: this.mouse.x, y: this.mouse.y };
      this.nodes = [...this.nodes, node];

      this.restartSimulation();
    },

restartSimulation: function() {
      this.nodesElem = this.nodesElem
        .data(this.nodes, d => d.index)
        .join("circle")
        .attr("r", this.nodeRadius)
        .attr("fill", this.nodeColor)
        .on("mousedown", this.onNodeMouseDown)
        .on("mouseup", this.onNodeMouseUp)
        .on("mouseover", this.onNodeMouseOver)
        .on("mouseout", this.onNodeMouseOut);

      this.linksElem = this.linksElem
        .data(this.links, d => [d.source.index, d.source.index])
        .join("line")
        .attr("stroke", this.linkColor)
        .attr("stroke-width", this.linkWidth);

      this.simulation.nodes(this.nodes);
      this.simulation.force(
        "link",
        d3.forceLink(this.links).id(d => d.index)
      );

      this.simulation.alpha(1).restart();
    },
    ticked: function() {
      this.nodesElem.attr("cx", d => d.x).attr("cy", d => d.y);

      this.linksElem
        .attr("x1", d => d.source.x)
        .attr("y1", d => d.source.y)
        .attr("x2", d => d.target.x)
        .attr("y2", d => d.target.y);

      this.mouselink = this.mouselink
        .data(this.isCreatingLink ? [this.nodeFrom] : [])
        .join("line")
        .attr("x1", this.mouse && this.mouse.x)
        .attr("y1", this.mouse && this.mouse.y)
        .attr("x2", d => d.x)
        .attr("y2", d => d.y)
        .attr("pointer-events", "none");
    }
  },
  mounted: function() {
    this.init();
  }
};
</script>

<style>
#context {
  height: 95vh;
  width: 95vw;
}
</style>