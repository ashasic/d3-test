<script setup>
import Chart from "./components/Charts.vue";
const message = "My D3 Charts";

</script>

<template>
  <div class="container-fluid">
    <div class="row">
      <div class="col">
        <h1 class="text-center mb-4">{{ message }}</h1>
      </div>
    </div>

    <div class="row">
      <div class="col">
        <Chart />
      </div>
    </div>
  </div>
</template>

