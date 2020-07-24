<template>
  <div class="fixed top-0 left-0 right-0 bottom-0 bg-black-60"
    @click.self="$emit('close')">
    <div class="absolute top-0 bottom-0 right-0 bg-white shadow-2 | slide-over">
      <div class="f6 tr bg-white">
        <a class="dib pa2 link hot-pink dim pointer" @click="$emit('close')">Close</a>
        <div ref="widget"></div>
      </div>
    </div>
  </div>
</template>

<script>
import { embed } from 'paypresto.js'

export default {
  methods: {
    mount(payment) {
      payment
        .mount(embed(this.$refs.widget))
    }
  }
}
</script>