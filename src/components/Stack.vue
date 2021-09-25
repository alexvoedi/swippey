<template>
  <div class="relative" :style="style">
    <Swipeable
      v-for="(slot, index) in $slots"
      :key="index" :ref="setItemRef"
      :queue-position="getQueuePosition(+index)"
      @swipe="(direction) => onSwipe(direction)"
    >
      <slot :name="index" :dragging="true"></slot>
    </Swipeable>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, onBeforeUpdate, ref } from 'vue'
import Swipeable from '@/components/Swipeable.vue'

export default defineComponent({
  components: {
    Swipeable,
  },

  props: {
    items: {
      type: Array,
      required: true,
    },
    size: {
      type: Object,
      default: () => ({
        width: 400,
        height: 600,
      })
    }
  },

  setup(props, { emit }) {
    let itemRefs: any[] = []

    const setItemRef = (el: any) => {
      if (el) {
        itemRefs.push(el)
      }
    }

    onBeforeUpdate(() => {
      itemRefs = []
    })

    const hasNext = computed(() => currentIndex.value < itemRefs.length)
    const hasPrevious = computed(() => currentIndex.value > 0)

    const currentIndex = ref(0)

    const next = (direction: number) =>
      currentIndex.value = Math.min(Math.max(currentIndex.value + direction, 0), itemRefs.length)

    const skip = () => {
      if (hasNext.value) {
        const item = itemRefs[currentIndex.value]
        item.swipeUp()

        next(+1)
      }
    }

    const undo = () => {
      if (hasPrevious.value) {
        next(-1)

        const item = itemRefs[currentIndex.value]
        item.reset()
      }
    }

    const getQueuePosition = (index: number) => index - currentIndex.value

    const onSwipe = (direction: any) => {
      emit('swipe', direction)
      next(+1)
    }

    const sizeCss = computed(() => ({
        width: `${props.size.width}px`,
        height: `${props.size.height}px`,
    }))

    const style = computed(() => [
      sizeCss.value
    ])

    return {
      getQueuePosition,
      setItemRef,
      onSwipe,
      currentIndex,
      next,
      skip,
      undo,
      style,
    }
  },
})
</script>

<style>
.relative {
  position: relative;
}

.absolute {
  position: absolute;
  top: 0;
}

.overflow-hidden {
  overflow: hidden;
}
</style>
