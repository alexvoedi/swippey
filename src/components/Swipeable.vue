<template>
  <div ref="draggable" class="absolute" :style="style">
    <slot></slot>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, onBeforeUnmount, onMounted, reactive, ref, provide } from 'vue'
import interact from "interactjs";

const INTERACT_ON_START = "start";
const INTERACT_ON_MOVE = "move";
const INTERACT_ON_END = "end";

const SWIPE_LEFT = "swipe-left";
const SWIPE_RIGHT = "swipe-right";
const SWIPE_UP = "swipe-up";
const SWIPE_DOWN = "swipe-down";
const SWIPE_ANY = "swipe";

export default defineComponent({
  name: 'Swipeable',

  props: {
    queuePosition: {
      type: Number,
      required: true,
    },
    maxRotation: {
      type: Number,
      default: 15,
    },
    outOfSightOffsetX: {
      type: Number,
      default: 500
    },
    outOfSightOffsetY: {
      type: Number,
      default: 500
    },
  },

  setup(props, ctx) {
    const draggable = ref()

    const transformation = reactive({
      x: 0,
      y: 0,
      rotation: 0,
    })

    const dragging = ref(false)
    const swiped = ref(false)

    const transformCss = computed(() => {
      const { x, y, rotation } = transformation;

      return {
        transform: `translate3D(${x}px, ${y - 10 * props.queuePosition}px, 0) rotate(${rotation}deg) scale(${1 - Math.abs(props.queuePosition) / 10})`
      };
    })

    const transitionCss = computed(() => {
      if (dragging.value) return {}

      return {
        transition: 'all 0.5s cubic-bezier(0.2, 0.8, 0.4, 1.0)'
      }
    })

    const opacityCss = computed(() => {
      if (swiped.value || props.queuePosition > 2) {
        return {
          opacity: 0
        }
      } else {
        return {
          opacity: 1
        }
      }
    })

    const zIndexCss = computed(() => {
      return {
        zIndex: 100 - props.queuePosition,
      }
    })

    const pointerEventsCss = computed(() => {
      return {
        pointerEvents: swiped.value ? 'none' : 'inherit'
      }
    })

    const style = computed(() => [
      transformCss.value,
      transitionCss.value,
      opacityCss.value,
      zIndexCss.value,
      pointerEventsCss.value,
    ])

    const setTransformation = (t: any) => {
      const { x = 0, y = 0, rotation = 0 } = t;

      Object.assign(transformation, { x, y, rotation })
    }

    const swipeRight = () => {
      swiped.value = true;

      const { maxRotation, outOfSightOffsetX } = props

      setTransformation({
        x: +outOfSightOffsetX,
        rotation: +maxRotation,
      });

      ctx.emit(SWIPE_RIGHT)

      interact(draggable.value).draggable(false)
    }

    const swipeLeft = () => {
      swiped.value = true;

      const { maxRotation, outOfSightOffsetX } = props

      setTransformation({
        x: -outOfSightOffsetX,
        rotation: -maxRotation,
      });

      ctx.emit(SWIPE_LEFT);

      interact(draggable.value).draggable(false)
    }

    const swipeUp = () => {
      swiped.value = true;

      const { outOfSightOffsetY } = props

      setTransformation({
        y: -outOfSightOffsetY,
        rotation: 0,
      });

      ctx.emit(SWIPE_UP);

      interact(draggable.value).draggable(false)
    }

    const swipeDown = () => {
      swiped.value = true;

      const { outOfSightOffsetY } = props

      setTransformation({
        y: +outOfSightOffsetY,
        rotation: 0,
      });

      ctx.emit(SWIPE_DOWN);

      interact(draggable.value).draggable(false)
    }

    const reset = () => {
      swiped.value = false

      setTransformation({
        x: 0,
        y: 0,
        rotation: 0,
      });

      interact(draggable.value).draggable(true)
    }

    const thresholdReached = (direction: any) => {
      ctx.emit(SWIPE_ANY, direction);

      switch (direction) {
        case SWIPE_RIGHT: {
          swipeRight()
          break
        }
        case SWIPE_LEFT: {
          swipeLeft()
          break;
        }
        case SWIPE_UP: {
          swipeUp()
          break;
        }
        case SWIPE_DOWN: {
          swipeDown()
          break;
        }
      }
    }

    const onstart = () => {
      ctx.emit(INTERACT_ON_START);
      dragging.value = true;
    }

    const onmove = (e: any) => {
      ctx.emit(INTERACT_ON_MOVE);

      const { maxRotation } = props
      const thresholdX = 200

      const x = transformation.x + e.dx
      const y = transformation.y + e.dy

      let rotation = maxRotation * (x / thresholdX)

      if      (rotation > +maxRotation) rotation = +maxRotation
      else if (rotation < -maxRotation) rotation = -maxRotation

      setTransformation({ x, y, rotation })
    }

    const onend = () => {
      ctx.emit(INTERACT_ON_END);
      dragging.value = false;

      const thresholdX = 200
      const thresholdY = 200

      const { x, y } = transformation

      if      (x > +thresholdX) thresholdReached(SWIPE_RIGHT)
      else if (x < -thresholdX) thresholdReached(SWIPE_LEFT)
      else if (y < -thresholdY) thresholdReached(SWIPE_UP)
      else if (y > +thresholdY) thresholdReached(SWIPE_DOWN)
      else                      reset()
    }

    onMounted(() => {
      interact(draggable.value).draggable({
        onstart,
        onmove,
        onend,
      })
    })

    onBeforeUnmount(() => {
       interact(draggable.value).unset()
    })

    provide('dragging', dragging)

    return {
      draggable,
      dragging,
      style,
      transformation,
      transformCss,
      reset,
      swipeLeft,
      swipeRight,
      swipeUp,
      swipeDown,
    }
  },
})
</script>

<style scoped>
.absolute {
  transform-origin: top;
  position: absolute;
  touch-action: none;
  width: 100%;
  height: 100%;
}
</style>
