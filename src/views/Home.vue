<template>
  <div class="home">
    <virtual-list
      v-if="list.length"
      :buffer="bufferSize"
      :list="list"
      :total="total"
      :height="800"
      :estimate-row-height="100"
    />
  </div>
</template>

<script>
// import faker from 'faker';
import VirtualList from '@/components/VirtualList.vue';

export default {
  name: `Home`,

  components: {
    VirtualList,
  },

  data() {
    return {
      bufferSize: 5,
      list: [],
    };
  },

  computed: {
    total({ list }) {
      return list.length;
    },
  },

  mounted() {
    for (let i = 0; i < 10000; i++) {
      // this.data.push(faker.loream.sentences());
      this.list.push({ index: i, content: i });
    }
  },
};
</script>

<style lang="scss" scoped>
.home {
  width: 100%;
  height: 90vh;
}
</style>
