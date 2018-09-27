<template>
    <div id="app">
        <div class="title">
            {{list.length}}Item
        </div>
        <CollectionRepeat :list="list">
            <template slot-scope="scope">
                <div class="my-item">
                    {{ scope.item }}
                </div>
            </template>
        </CollectionRepeat>
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

<style>
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
    }

    body {
        margin: 0;
    }

    .title {
        height: 50px;
        line-height: 50px;
        background-color: #387ef5;
        color: #fff;
    }

    .my-item {
        border-top: 1px #ddd solid;
        border-bottom: 1px #ddd solid;
        height: 55px;
        line-height: 55px;
    }
</style>
