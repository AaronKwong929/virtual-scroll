<template>
  <div class="home">
    <!-- <virtual-list
      v-if="list && list.length > 0"
      :buffer="bufferSize"
      :list="list"
      :total="total"
      :height="800"
      :estimate-row-height="100"
    /> -->
    <virtual-list
      v-if="list && list.length > 0"
      :buffer="bufferSize"
      :list="list"
      :total="total"
      :height="600"
    />
  </div>
</template>

<script>
import faker from 'faker';
import VirtualList from '@/components/VirtualList2.vue';

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
    const list = [];
    for (let i = 0; i < 10000; i++) {
      // this.data.push(faker.loream.sentences());
      list.push({ index: i, content: faker.lorem.sentences() });
    }
    this.list = list;
  },
};
</script>

<style lang="scss" scoped>
.home {
  width: 100%;
  height: 90vh;
}
</style>
