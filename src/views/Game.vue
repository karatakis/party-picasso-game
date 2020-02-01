<template>
  <div class="about">
    <h1>Game Canvas</h1>
    <!-- <demo /> -->
    <draw-view />
  </div>
</template>

<script>
// import Demo from '@/components/Demo.vue'
import DrawView from '@/components/DrawView.vue'

export default {
  name: 'GamePage',
  components: {
    DrawView
    // Demo
  }
}
</script>
