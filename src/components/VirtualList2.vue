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
    />

    <div
      ref="actualContent"
      class="actual-content"
      :style="{ position: `absolute` }"
    >
      <div
        v-for="item in renderContentList"
        :key="`list-item-${item.index}`"
        :style="{
          width: `100%`,
          height: `${estimateRowHeight}px`,
          position: `absolute`,
          left: 0,
          right: 0,
          top: `${item.index * estimateRowHeight}px`,
          borderBottom: `1px solid #000`,
        }"
      >
        {{ item.content }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: `VirtualList`,

  props: {
    list: {
      type: Array,
      required: true,
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

  data() {
    return {
      startIndex: 0,
      originalStartIndex: 0,
      endIndex: 0,
      scrollTop: 0,

      cachedPositions: [],
      phantomHeight: 0,
    };
  },

  computed: {
    limit({ height, estimateRowHeight }) {
      return Math.ceil(height / estimateRowHeight);
    },

    renderContentList({ startIndex, endIndex, list }) {
      return list.slice(startIndex, endIndex);
    },
  },

  methods: {
    handleScroll(event) {
      if (event.target === this.$refs[`scrollingContainer`]) {
        const {
          target: { scrollTop },
        } = event;

        const currentStartIndex = Math.floor(
          scrollTop / this.estimateRowHeight
        );
        if (currentStartIndex !== this.startIndex) {
          this.originalStartIndex = currentStartIndex;

          this.startIndex = Math.max(this.originalStartIndex - this.buffer, 0);
          console.log(`start index`, this.startIndex);
          this.endIndex = Math.min(
            currentStartIndex + this.limit,
            this.total - 1
          );

          this.scrollTop = scrollTop;
        }
      }
    },

    getTransform() {
      const { scrollTop } = this;
      const { estimateRowHeight, buffer, originalStartIndex } = this;
      return `translate3d(0,${scrollTop -
        (scrollTop % estimateRowHeight) -
        Math.min(originalStartIndex, buffer) * estimateRowHeight}px,0)`;
    },

    initCachedPositions() {
      const { estimateRowHeight, list } = this;
      this.cachedPositions = [];
      for (let i = 0; i < list.length; i++) {
        this.cachedPositions.push({
          index: i,
          height: estimateRowHeight,
          top: i * estimateRowHeight,
          bottom: (i + 1) * estimateRowHeight,
          dValue: 0,
        });
      }
      this.estimateRowHeight = this.cachedPositions[
        this.cachedPositions.length - 1
      ].bottom; // 总高度等于数组最后一个元素的 bottom 数值
    },

    updateCachedPositions() {
      
    },
  },

  updated() {
    if (this.$refs[`actualContent`] && this.total) {
      this.updateCachedPositions();
    }
    return;
  },

  mounted() {
    this.endIndex = Math.min(
      this.startIndex + this.limit + this.buffer,
      this.total - 1
    );
    this.initCachedPositions();
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
