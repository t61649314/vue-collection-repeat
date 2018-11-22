<template>
  <div id="app">
    <div class="title">
      {{list.length}}Item
    </div>
    <div class="collection-box">
      <CollectionRepeat :list="list">
        <template slot-scope="scope">
          <div class="my-item">
            {{ scope.item }}
          </div>
        </template>
      </CollectionRepeat>
    </div>
  </div>
</template>

<script>
  import CollectionRepeat from './components/collectionRepeat.vue'

  export default {
    name: 'App',
    components: {
      CollectionRepeat
    },
    data() {
      return {
        list: []
      }
    },
    mounted() {
      for (let i = 1; i < 100000; i++) {
        this.list.push(i);
      }
    }
  }
</script>

<style lang="less">
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    height: 100%;
    font-size: 16px;
  }

  body {
    margin: 0;
  }

  .title {
    height: 1rem;
    line-height: 1rem;
    background-color: #387ef5;
    color: #fff;
  }

  .collection-box {
    position: relative;
    height: calc(100% - 1rem);
    .my-item {
      border-top: 0.01rem #ddd solid;
      border-bottom: 0.01rem #ddd solid;
      height: 1rem;
      line-height: 1rem;
    }
  }

</style>
