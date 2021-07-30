<template>
  <div
    ref="scrollingContainer"
    class="virtual-list"
    :style="{ height: `${height}px` }"
    @scroll="handleScroll"
  >
    <div
      ref="phantomContainer"
      :style="{
        height: `${phantomHeight}px`,
        position: `relative`,
        zIndex: `-1`,
      }"
    />

    <div
      ref="actualContent"
      class="actual-content"
      :style="{
        position: `absolute`,
        transForm: getTransform(),
        top: 0,
        width: `100%`,
      }"
    >
      <div
        v-for="item in renderContentList"
        :key="`list-item-${item.index}`"
        :id="`list-item-${item.index}`"
        :style="{
          width: `100%`,
          height: `${estimateRowHeight}px`,
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

    // estimateRowHeight: {
    //   type: Number,
    //   required: true,
    // },

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
      estimateRowHeight: 0,
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

  watch: {
    list() {
      if (this.$refs[`actualContent`] && this.total) {
        this.updateCachedPositions();
      }
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
      ].bottom; // NOTE: 总高度等于数组最后一个元素的 bottom 数值
    },

    updateCachedPositions() {
      const nodeList = this.$refs[`actualContent`].childNodes;
      const start = nodeList[0];
      nodeList.forEach(node => {
        if (!node) return;
        const { height } = node.getBoundingClientRect();
        const index = Number(node.id.split(`-`)[2]);
        const oldHeight = this.cachedPositions[index].height;
        const dValue = oldHeight - height;

        if (dValue) {
          this.cachedPositions[index].bottom -= dValue;
          this.cachedPositions[index].height = height;
          this.cachedPositions[index].dValue = dValue;
        }
      });

      // eslint-disable-next-line no-unused-vars
      let startIndex = 0;
      if (start) startIndex = Number(start.id.split(`-`)[2]);
      const { length: cachedPositionsLen } = this.cachedPositions;
      let cumulativeDiffHeight = this.cachedPositions[startIndex].dValue;
      this.cachedPositions[startIndex].dValue = 0;

      for (let i = startIndex + 1; i < cachedPositionsLen; ++i) {
        const item = this.cachedPositions[i];
        // update height
        this.cachedPositions[i].top = this.cachedPositions[i - 1].bottom;
        this.cachedPositions[i].bottom =
          this.cachedPositions[i].bottom - cumulativeDiffHeight;

        if (item.dValue !== 0) {
          cumulativeDiffHeight += item.dValue;
          item.dValue = 0;
        }
      }

      // update our phantom div height
      const height = this.cachedPositions[cachedPositionsLen - 1].bottom;
      this.phantomHeight = height;
      // this.$refs[`phantomContainer`].style.height = `${height}px`;
    },
  },

  updated() {},

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
