<template>
  <div
    ref="scrollingContainer"
    class="virtual-list"
    :style="{ height: `${height}px` }"
    @scroll="handleScroll"
  >
    <div
      ref="phantomContainer"
      :style="{ height: `${phantomHeight}px`, position: `relative` }"
    >
      {{  }}
    </div>
  </div>
</template>

<script>
export default {
  name: `VirtualList`,

  props: {
    list: {
      type: Array,
      default: () => [],
    },

    height: {
      type: Number,
      required: true,
    },

    total: {
      type: Number,
      required: true,
    },

    estimateRowHeight: {
      type: Number,
      required: true,
    },

    buffer: {
      type: Number,
      default: 5,
    },
  },

  computed: {
    phantomHeight({ estimateRowHeight, total }) {
      return estimateRowHeight * total;
    },

    limit({ height, estimateRowHeight }) {
      return Math.ceil(height / estimateRowHeight);
    },
  },

  watch: {},

  methods: {
    handleScroll() {},
  },
};
</script>

<style lang="scss" scoped>
.virtual-list {
  overflow-y: auto;
  overflow-x: hidden;
  position: relative;
}
</style>
