<script setup>
import Barchart from '@/components/BarChart.vue'

</script>
<template>
  <div>
    <!-- <nav>
      <router-link to="/">
        Home
      </router-link> |
      <router-link to="/about">
        About
      </router-link>
    </nav>
    <router-view /> -->
    <div class="container">
      <h2 class="text-center">
        D3 example
      </h2>
      <div class="row">
        <div class="col-md-6">
          <Barchart />
        </div>
        <div class="col-md-6">
          第二個
        </div>
      </div>
    </div>
  </div>
</template>
