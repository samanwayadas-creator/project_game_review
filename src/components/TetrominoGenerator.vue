<template>
  <img
    :src="require('../assets/tetrominos/' + color + '.png')"
    alt="tetromino"
    draggable="false"
  />
</template>

<script>
export default {
  name: "TetrominoGenerator",
  props: {
    color: {
      type: String,
      required: true,
    },
  },
};
</script>

<style>
img {
  width: 2rem;
  margin: 0px;
  padding: 0px;
  display: block;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
