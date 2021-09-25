<template>
  <div class="card" :class="{ dragging }">
    <img class="card-image" :src="data.download_url" loading="lazy">
  </div>
</template>

<script lang="ts">
import { defineComponent, inject } from 'vue'

export default defineComponent({
  props: {
    data: {
      type: Object,
      required: true,
    },
  },

  setup() {
    const dragging = inject('dragging', false)

    return {
      dragging,
    }
  },
})
</script>

<style scoped>
.card {
  background-color: white;
  position: relative;
  overflow: hidden;
  width: 100%;
  height: 100%;
  border-radius: 16px;
  transition: all 0.25s ease;
  will-change: transform;
  box-shadow: rgba(0, 0, 0, 0.25) 0px 5px 5px;
}

.dragging {
  transform: scale(1.05);
  box-shadow: rgba(0, 0, 0, 0.5) 0px 5px 15px;
}

.card-image {
  object-fit: cover;
  overflow: hidden;
  height: 100%;
  width: 100%;
}
</style>
