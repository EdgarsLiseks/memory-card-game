<template>
  <div v-if="card.matched" class="card matched" />

  <div v-else class="card" @click="clickHandler">
    <img v-if="card.open" :src="card.image" />
    <img v-else class="card-closed" src="@/assets/back_card.png" />
  </div>
</template>

<script>
export default {
  props: {
    card: Object
  },
  data: () => {
    return {};
  },
  methods: {
    clickHandler: function() {
      if (!this.card.open && !this.card.matched) {
        this.$parent.$emit("onCardOpen", this.card);
      }
    }
  }
};
</script>

<style>
.card {
  user-select: none;
  cursor: pointer;
}
.card-closed {
  max-height: 50px;
}
.card > img {
  max-height: 50px;
}

@media (min-width: 768px) {
  .card-closed {
  max-height: 100px;
}
.card > img {
  max-height: 100px;
}
}
</style>