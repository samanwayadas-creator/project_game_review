<template>
  <Keypress
    key-event="keydown"
    :key-code="39"
    @success="$emit('rightLeftFunc', 'right')"
  />
  <Keypress
    key-event="keydown"
    :key-code="37"
    @success="$emit('rightLeftFunc', 'left')"
  />
  <Keypress
    key-event="keydown"
    :key-code="38"
    @success="$emit('rotateFunc', 'rotate')"
  />
  <Keypress
    key-event="keydown"
    :key-code="40"
    @success="$emit('downFunc', 'down')"
  />
</template>

<script>
import { defineAsyncComponent } from "vue";

export default {
  name: "KeyMapped",
  components: {
    Keypress: defineAsyncComponent(() => import("vue-keypress")),
  },
  emits: ["rightLeftFunc", "downFunc", "rotateFunc"],
};
</script>
