<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
import Text from './components/Text.vue'
import CoffeeCup from './components/CoffeeCup.vue'
</script>

<template>
  <div class="container">
    <div class="row">
      <div class="one-third column">&nbsp;</div>
      <div class="one-third column">
        <CoffeeCup />
      </div>
      <div class="one-third column">&nbsp;</div>
    </div>
    <div class="row">
      <div class="column">
        <Text />
      </div>
    </div>
  </div>
</template>

<style>
@import "./assets/skeleton.css";

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
