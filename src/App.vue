<template>
  <div id="app">
    <ForceGraph
      nodeColor="black"
      v-bind:inputNodes="[{x: 154, y: 542}, {x: 132, y: 123}]"
      v-bind:inputLinks="[{source: 0, target: 1}]"
    />
  </div>
</template>

<script>
import ForceGraph from './components/ForceGraph.vue'
export default {
  name: 'App',
  components: {
    ForceGraph
  }
}
</script>
