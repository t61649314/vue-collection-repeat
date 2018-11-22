<template>
  <div ref="listContent" class="collection-repeat-content"
       @touchstart="touchStart"
       @touchmove="touchMove"
       @touchend="touchEnd">
    <div class="scroll" v-if="showList.length"
         :style="`transform: translate3d(0px, ${scrollTranslate3dY}px, 0px) scale(1);`">
      <div class="list disable-user-behavior">
        <div class="item" v-for="(item,index) in showList"
             :style="`transform: translate3d(0px, ${item.translate3dY}px, 0px) scale(1);`">
          <slot v-bind:item="item.data" v-bind:index="index"></slot>
        </div>
      </div>
    </div>
    <div class="hide-item" ref="hideItem" v-if="list.length&&!getHeightDone">
      <slot v-bind:item="list[0]"></slot>
    </div>
  </div>
</template>

<script>

  export default {
    name: 'CollectionRepeat',
    props: {
      list: {
        type: Array,
        default: function () {
          return [];
        }
      },
      margin: {
        type: Number,
        default: 0
      }
    },
    data() {
      return {
        minScrollTranslate3dY: 0,
        getHeightDone: false,
        lastIndex: 0,
        firstIndex: 0,
        onChangeY: 0,
        itemHeight: 0,
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
      },
      scrollTranslate3dY: function (newVal, oldVal) {
        //往下滚动
        if (newVal < oldVal) {
          if (newVal < this.onChangeY && this.lastIndex < this.mList.length - 1) {
            let pushCount = parseInt((this.onChangeY - newVal) / this.itemHeight);
            for (let i = 0; i < pushCount; i++) {
              this.onChangeY -= this.itemHeight;
              this.firstIndex++;
              this.lastIndex++;
              this.showList.push(this.mList[this.lastIndex]);
              this.showList.shift();
            }
          }
        } else {
          if (newVal < 0 && newVal > this.onChangeY && this.firstIndex > 0) {
            let pushCount = parseInt((newVal - this.onChangeY) / this.itemHeight);
            for (let i = 0; i < pushCount; i++) {
              this.onChangeY += this.itemHeight;
              this.firstIndex--;
              this.lastIndex--;
              this.showList.unshift(this.mList[this.firstIndex]);
              this.showList.pop();
            }
          }
        }
      }
    },
    mounted: function () {
      this.windowHeight = window.innerHeight;
    },
    methods: {
      drawList: function () {
        this.getHeightDone = false;
        this.$nextTick(() => {
          const length = this.mList.length;
          this.itemHeight = this.$refs.hideItem.clientHeight - 1 + this.margin;
          this.minScrollTranslate3dY = this.windowHeight - length * this.itemHeight;
          this.getHeightDone = true;
          const windowCoverCount = parseInt(this.windowHeight * 1.6 / this.itemHeight);
          const count = length < windowCoverCount ? length : windowCoverCount;
          this.lastIndex = count - 1;
          this.firstIndex = 0;
          this.onChangeY = -this.windowHeight * 0.3;
          this.mList = this.mList.map((item, index) => {
            return {
              data: item,
              translate3dY: index * this.itemHeight
            }
          });
          this.showList = this.mList.slice(0, count)
        });
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
        //如果向下拉动使顶部脱离顶部或者向上拉动使底部触底则只能移动一半的距离
        if ((moveY > 0 && this.scrollTranslate3dY > 0) || (moveY < 0 && this.scrollTranslate3dY < this.minScrollTranslate3dY)) {
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


        if (this.scrollTranslate3dY < 0 && this.scrollTranslate3dY > this.minScrollTranslate3dY) {
          //结束拉动发现顶部正常且没有在结束前骤停，则继续惯性滑动
          if (lastMoveTime < 300) {
            this.inertiaMove(v, startTime, this.scrollTranslate3dY);
          }
        } else {
          //结束拉动发现顶部脱离则回到顶部
          this.topScrollBack();
        }
      },
      topScrollBack: function () {
        if (this.stopInertiaMove) {
          return;
        }
        let limit = this.scrollTranslate3dY > 0 ? 0 : this.minScrollTranslate3dY;
        //回弹速度为Y轴偏移量*0.04（暂定）
        let backV = (this.scrollTranslate3dY - limit) * 0.04;
        let backY = this.scrollTranslate3dY - backV;
        this.scrollTranslate3dY = backY;

        //接近临界值就强行停止，不然会一直执行
        if (Math.round(backY) !== limit) {
          setTimeout(() => {
            this.topScrollBack();
          });
        } else {
          this.scrollTranslate3dY = limit;
        }
      },
      inertiaMove: function (v, startTime, contentY) {
        const nowTime = Date.now();
        if (this.stopInertiaMove) {
          return;
        }
        //加速度方向
        const dir = v > 0 ? -1 : 1;
        const deceleration = dir * 0.0006;
        const t = nowTime - startTime;
        //速度持续靠近0
        const nowV = v + t * deceleration;
        // 速度方向变化表示速度达到0了，则停止
        if (dir * nowV > 0) {
          return;
        }
        const moveY = (v + nowV) / 2 * t;
        this.scrollTranslate3dY = contentY + moveY;

        // 惯性运动过程中使顶部脱离，并且还存在持续的速度，则让这个速度*100（暂定）为回滚的临界点
        if ((this.scrollTranslate3dY > 0 && this.scrollTranslate3dY > nowV * 100)
          || (this.scrollTranslate3dY < this.minScrollTranslate3dY && this.scrollTranslate3dY - this.minScrollTranslate3dY < nowV * 100)) {
          setTimeout(() => {
            this.topScrollBack()
          });
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
  .collection-repeat-content {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    overflow: hidden;
    margin-top: -1px;
    padding-top: 1px;
    margin-bottom: -1px;
    width: auto;
    height: auto;
    .hide-item {
      visibility: hidden;
    }

    .list {
      position: relative;
      padding-top: 1px;
      padding-bottom: 1px;
      .item {
        width: 100%;
        box-sizing: border-box;
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
      -webkit-user-drag: none;
      -ms-touch-action: none;
      -ms-content-zooming: none;
    }
  }
</style>
