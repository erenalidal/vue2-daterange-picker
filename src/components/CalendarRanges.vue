<template>
  <div class="ranges">
    <ul v-if="ranges" class="hidden-xs">
      <li
        v-for="(value, key) in ranges"
        @click="$emit('clickRange', value)"
        :class="rangeClasses(value)"
        :data-range-key="key"
        :key="key"
      >{{key}}
      </li>
    </ul>
  </div>
</template>

<script>
  export default {
    props: ['ranges', 'preset'],
    methods: {
      rangeClasses(value) {
        console.log(this.preset, value)
        return {'active-range': this.preset === value[2]};
      }
    },
  }
</script>

<style scoped lang="scss">
  .active-range {
    border: 1px solid;
    background-color: #357ebd;
    color: #ffffff;

    &:hover {
      color: #000000;
    }
  }
</style>
