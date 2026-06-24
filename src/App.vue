<script setup>
import AgentDashboard from './components/AgentDashboard.vue'
</script>

<template>
  <div id="app">
    <AgentDashboard />
  </div>
</template>

<style>
:root {
  color-scheme: dark;
  color: #edf2ff;
  background-color: #11141f;
}

html,
body,
#app {
  min-height: 100%;
  min-height: 100vh;
  margin: 0;
  padding: 0;
  background:
    radial-gradient(circle at top left, rgba(0, 242, 254, 0.12), transparent 28%),
    linear-gradient(180deg, #11141f 0%, #0f1220 50%, #0a0d17 100%);
  font-family: 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

body {
  overflow-x: hidden;
}

#app {
  display: flex;
  min-height: 100vh;
}
</style>
