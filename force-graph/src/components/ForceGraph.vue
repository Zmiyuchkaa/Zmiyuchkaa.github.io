<template>
  <svg id="context" />
</template>

<script>
import * as d3 from "d3";

export default {
  name: "ForcedGraph",
  data: () => ({
    // nodes: [],
    // links: [],
  }),
  methods: {
    init: function() {
      let nodes = [];
      let links = [];

      let width = 500;
      let height = 500;
      let minDistanceToConnect = 100;
      let nodeColor = "black";
      let nodeRadius = 7;
      let mouse = null;

      let isCreatingLink = false;
      let nodeFrom = null;
      let nodeTo = null;

      const inrange = (source, target) =>
        Math.hypot(source.x - target.x, target.y - target.y) !== 0;

      const svg = d3
        .select("#context")
        .property("value", { nodes: [], links: [] })
        .attr("viewBox", [-width / 2, -height / 2, width, height])
        .on("mousedown", onSvgMouseDown)
        .on("mousemove", onSvgMouseMove)
        .on("mouseup", onSvgMouseUp);

      // links
      let linksElem = svg.append("g").selectAll("line");

      // nodes
      let nodesElem = svg.append("g").selectAll("circle");

      // temp link line
      let mouselink = svg
        .append("g")
        .attr("stroke", "red")
        .selectAll("line");

      const simulation = d3
        .forceSimulation(nodes)
        .force(
          "link",
          d3.forceLink(links).id(d => d.index)
        )
        .force("charge", d3.forceManyBody().strength(-60))
        .force("x", d3.forceX())
        .force("y", d3.forceY())
        .on("tick", ticked);

      function onNodeMouseDown(d, i, svgNodes) {
        d3.event.stopPropagation();
        if (isCreatingLink === false) {
          isCreatingLink = true;
          nodeFrom = nodes[i];
        }
      }

      function onNodeMouseUp(d, i, svgNodes) {
        d3.event.stopPropagation();
        if (isCreatingLink === true) {
          nodeTo = nodes[i];
          if (nodeTo == null) {
            return;
          }

          if (nodeTo.index === nodeFrom.index) {
            nodeFrom = null;
            isCreatingLink = false;
            nodeTo = null;
            nodes.splice(i, 1);

            links = links.filter(function(l) {
              return l.source !== d && l.target !== d;
            });

            nodesElem = nodesElem
              .data(nodes)
              .join("circle")
              .attr("r", nodeRadius)
              .attr("color", nodeColor)
              .on("mousedown", (d, i) =>
                onNodeMouseDown(d, i, nodesElem, simulation)
              );

            linksElem = linksElem
              .data(links, d => [d.source.index, d.source.index])
              .join("line")
              .attr("stroke", "red")
              .attr("stroke-width", 1.5);

            simulation.nodes(nodes);
            simulation.force(
              "link",
              d3.forceLink(links).id(d => d.index)
            );

            simulation.alpha(1).restart();
            return;
          }

          links = [...links, { source: nodeFrom, target: nodeTo }];

          linksElem = linksElem
            .data(links, d => [d.source.index, d.source.index])
            .join("line")
            .attr("stroke", "red")
            .attr("stroke-width", 1.5);

          simulation.nodes(nodes);
          simulation.force(
            "link",
            d3.forceLink(links).id(d => d.index)
          );

          simulation.alpha(1).restart();
          isCreatingLink = false;
          nodeFrom = null;
          nodeTo = null;
        }
      }

      function onNodeMouseOver(d, i, svgNodes) {
        d3.select(this).attr("fill", "orange");
      }

      function onNodeMouseOut(d, i, svgNodes) {
        d3.select(this).attr("fill", nodeColor);
      }

      function onSvgMouseMove(d, i, svgNodes) {
        const [x, y] = d3.mouse(this);
        mouse = { x, y };
        simulation.alpha(0.3).restart();
      }

function onSvgMouseUp(d, i, svgNodes) {
        console.log("svg mouseup");
        if (isCreatingLink) {
          isCreatingLink = false;
          nodeFrom = null;
          nodeTo = null;
        }
      }

      function onSvgMouseDown(d, i, svgNodes) {
        d3.event.stopPropagation();
        const [x, y] = d3.mouse(svgNodes[i]);
        const node = { x, y };
        nodes = [...nodes, node];

        nodesElem = nodesElem
          .data(nodes, d => d.index)
          .join("circle")
          .attr("r", nodeRadius)
          .attr("color", nodeColor)
          .on("mousedown", onNodeMouseDown)
          .on("mouseup", onNodeMouseUp)
          .on("mouseover", onNodeMouseOver)
          .on("mouseout", onNodeMouseOut);

        simulation.nodes(nodes);

        simulation.alpha(1).restart();
      }

      function ticked() {
        nodesElem.attr("cx", d => d.x).attr("cy", d => d.y);

        linksElem
          .attr("x1", d => d.source.x)
          .attr("y1", d => d.source.y)
          .attr("x2", d => d.target.x)
          .attr("y2", d => d.target.y);

        mouselink = mouselink
          .data(isCreatingLink ? [nodeFrom] : [])
          .join("line")
          .attr("x1", mouse && mouse.x)
          .attr("y1", mouse && mouse.y)
          .attr("x2", d => d.x)
          .attr("y2", d => d.y)
          .attr("pointer-events", "none");
      }
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