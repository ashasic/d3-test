<script setup>
import Chart from "./components/Charts.vue";
const message = "Hello from Naimul!";

</script>

<template>
  <div class="container">

    <div class="row">
      <div class="col">
        <h1>{{ message }}</h1>
      </div>
    </div>

    <hr />

    <div class="row">
      <div class="col">
        <Chart />
      </div>
    </div>

    <hr />

    <div class="row">
      <div class="col">
        <p>This could be a footer</p>
      </div>
    </div>
  </div>
</template>

