<template>
    <div>
        <div class="title">
        </div>
        <div class="list-content" @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd">
            <div class="scroll" :style="`transform: translate3d(0px, ${scrollTranslate3dY}px, 0px) scale(1);`">
                <div class="list disable-user-behavior">
                    <div class="item" v-for="(item,index) in showList"
                         :style="`transform: translate3d(0px, ${index*54}px, 0px) scale(1);`">
                        {{ item }}
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'CollectionRepeat',
        props: {
            list: Array
        },
        data() {
            return {
                scrollTranslate3dY: 0,
                showList: [],
                mList: [],
                lastMoveStart: 0,
                lastMoveTime: 0,
                stopInertiaMove: false,
            }
        },
        watch: {
            list: function (newVal, oldVal) {
                this.mList = newVal;
                this.drawList();
            }
        },
        mounted: function () {
        },
        methods: {
            drawList: function () {
                this.showList = this.mList.slice(0, 20);
            },
            touchStart: function (e) {
                this.lastY = e.touches[0].pageY;

                /**
                 * 缓动代码
                 */
                this.lastMoveStart = this.lastY;
                this.lastMoveTime = Date.now();
                this.stopInertiaMove = true;
            },
            touchMove: function (e) {
                let nowY = e.touches[0].pageY;
                let moveY = nowY - this.lastY;
                if (this.scrollTranslate3dY > 0 && moveY > 0) {
                    this.scrollTranslate3dY += (moveY / 2);
                } else {
                    this.scrollTranslate3dY += moveY;
                }
                this.lastY = nowY;
                /**
                 * 缓动代码
                 */
                let nowTime = Date.now();
                this.stopInertiaMove = true;
                if (nowTime - this.lastMoveTime > 300) {
                    this.lastMoveTime = nowTime;
                    this.lastMoveStart = nowY;
                }

            },
            touchEnd: function (e) {
                /**
                 * 缓动代码
                 */
                const startTime = Date.now();
                this.stopInertiaMove = false;
                const lastMoveTime = startTime - this.lastMoveTime;
                const v = (this.lastY - this.lastMoveStart) / lastMoveTime; //最后一段时间手指划动速度
                if (this.scrollTranslate3dY < 0) {
                    if (lastMoveTime < 300) {
                        this.inertiaMove(v, startTime, this.scrollTranslate3dY);
                    }
                } else {
                    this.topScrollBack();
                }
            },
            topScrollBack: function (nowV) {
                if (this.stopInertiaMove) {
                    return;
                }
                let backV = this.scrollTranslate3dY * 0.04;
                let backY = this.scrollTranslate3dY - backV;
                if (backY < 0) {
                    backY = 0;
                }
                this.scrollTranslate3dY = backY;
                if (backY !== 0) {
                    setTimeout(() => {
                        this.topScrollBack();
                    }, 10);
                }
            },
            inertiaMove: function (v, startTime, contentY) {
                const nowTime = Date.now();
                if (this.stopInertiaMove) {
                    return;
                }
                const dir = v > 0 ? -1 : 1; //加速度方向
                const deceleration = dir * 0.0006;
                const t = nowTime - startTime;
                const nowV = v + t * deceleration;
                // 速度方向变化表示速度达到0了
                if (dir * nowV > 0) {
                    return;
                }
                const moveY = (v + nowV) / 2 * t;
                this.scrollTranslate3dY = contentY + moveY;
                if (this.scrollTranslate3dY > 0) {
                    setTimeout(() => {
                        this.topScrollBack()
                    }, 10);

                } else {
                    setTimeout(() => {
                        this.inertiaMove(v, startTime, contentY)
                    }, 10);
                }
            }
        }
    }
</script>

<style lang="less" rel="stylesheet/less" scoped>
    .list-content {
        position: absolute;
        top: 50px;
        right: 0;
        bottom: 0;
        left: 0;
        overflow: hidden;
        margin-top: -1px;
        padding-top: 1px;
        margin-bottom: -1px;
        width: auto;
        height: auto;
    }

    .title {
        height: 50px;
        background-color: red;
    }

    .list {
        position: relative;
        padding-top: 1px;
        padding-bottom: 1px;
        .item {
            width: 100%;
            box-sizing: border-box;
            height: 55px;
            line-height: 21px;
            background-color: #fff;
            color: #444;
            display: block;
            margin: -1px;
            padding: 16px;
            border: 1px solid #ddd;
            font-size: 16px;
            left: 0 !important;
            top: 0 !important;
            position: absolute !important;
            z-index: 1;
        }
    }

    .disable-user-behavior {
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
        -webkit-touch-callout: none;
        -webkit-tap-highlight-color: transparent;
        -webkit-tap-highlight-color: transparent;
        -webkit-user-drag: none;
        -ms-touch-action: none;
        -ms-content-zooming: none;
    }
</style>
