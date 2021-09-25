<template>
  <div class="app-container">
    <Stack :items="cardData" ref="stack" @swipe="swipe">
      <template v-for="(data, index) in cardData" v-slot:[index] :key="index">
        <Card :data="data">
        </Card>
      </template>
    </Stack>

    <div>
      <button @click="undo()">Undo</button>
      <button @click="skip()">Skip</button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import Card from '@/components/Card.vue'
import Stack from '@/components/Stack.vue'
import axios from 'axios'

export default defineComponent({
  components: {
    Card,
    Stack,
  },

  setup() {
    const stack = ref()

    const cardData = ref([])

    const fetchImages = async () => {
      const { data } = await axios.get('https://picsum.photos/v2/list?limit=10')

      cardData.value = data
    }

    fetchImages()

    const undo = () => {
      stack.value.undo()
    }

    const skip = () => {
      stack.value.skip()
    }

    const swipe = (x: any) => {
      //
    }

    return {
      cardData,
      skip,
      undo,
      swipe,
      stack,
    }
  }
});
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto&display=swap');

html, body {
  padding: 0;
  margin: 0;
}

.app-container {
  font-family: 'Roboto', sans-serif;
  width: 100vw;
  height: 100vh;
  padding: 0;
  margin: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
