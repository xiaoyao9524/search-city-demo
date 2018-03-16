<template>
  <div class="search-list">
    <!--搜索框-->
    <section class="search">
      <button class="search-btn">搜索</button>
      <input type="text" class="search-input" @input="filterData($event)">
    </section>
    <!--定位城市-->
    <!--主列表-->
    <section ref="searchListWrapper"
             class="search-list-wrapper"
    >
      <ul class="search-list" ref="searchList">
        <li class="list-item"
            v-for="(item, key, index) in searchListCopy"
            ref="searchListItem"
        >
          <h3>{{key}}</h3>
          <ul class="child">
            <li class="child-item" v-for="(child, index2) in item"
                @click.stop.prevent="isSelectCity(child)"
            >
              {{child.cityName}}
            </li>
          </ul>
        </li>
      </ul>
    </section>
    <!--右侧首字母列表-->
    <section class="right-list-wrapper">
      <ul class="right-list"
          @touchstart.stop.prevent="isTouchStart($event)"
          @touchmove.stop.prevent="isTouchMove($event)"
          @touchend.stop.prevent="isTouchEnd($event)"
      >
        <li class="list-item"
            v-for="(item, key, index) in searchList"
            :class="{'current': currentElIndex === index}"
            ref="rightListItem"
            :data-index="index"
        >{{key}}
        </li>
      </ul>
    </section>
  </div>
</template>

<script>
  import BScroll from 'better-scroll'

  export default {
    name: 'search-list',
    props: {
      searchList: {
        type: Object,
        default: {}
      }
    },
    data() {
      return {
        msg: 'BaseList',
        searchListCopy: null, // 主要的列表数据
        heightArray: [], // 存放主列表所有元素的高度
        currentScrollIndex: 0,
        currentElIndex: 0,
        touchList: {
          isTouch: false, // 手指是否按下
          touchStartY: -1, // 手指按下的位置
          touchMoveY: -1, // 手指正在移动的位置
          touchEndY: -1 // 手指抬起的位置
        },
        rightItemHeight: 0, // 右侧每一项的高度
        scroll: null, // better-scroll的实例
        pageY: 0,  // 元素滚动的位置
        cityArr: [] // 存放所有城市的数组，方便汉字搜索
      }
    },
    mounted() {
      this.$refs.searchListWrapper.style.height = window.innerHeight - 24 + 'px';

    },
    methods: {
      isSelectCity(item) {
        this.$emit("selectCity", item);
      },
      filterData(ev) {
        this.searchListCopy = JSON.parse(JSON.stringify(this.searchList));
        let val = ev.target.value;
        if (!val) {
          return;
        }
        // 输入字母
        if (/^[a-zA-Z]+$/.test(val)) {
          // 只输入一个字母
          let obj = {};
          let initials = val.charAt(0);
          obj[initials] = this.searchListCopy[initials];
          this.searchListCopy = obj;
          if (val.length === 1) {
            return;
          }
          obj[initials] = obj[initials].filter(function (a) {
            // return a.Initials.indexOf(val) >= 0; // 使用indexof方法
            let re = new RegExp(val); // 使用正则
            // console.log(a.Initials.search(re))
            return a.Initials.search(re) >= 0;
          })
          this.searchListCopy = obj;
        }
        // 输入汉字
        if (/[\u4e00-\u9fa5]+/.test(val)) {

        }
      },
      jumpSearchListItem(index) { // 跳到某一个首字母对应的位置
        let top = this.heightArray[index];
        this.scroll.scrollTo(0, -top)
      },
      isTouchStart(ev) {
        this.touchList.isTouch = true;

        let firstTouch = ev.touches[0];
        let target = ev.target;
        this.targetIndex = parseInt(target.dataset.index);
        let top = this.heightArray[this.targetIndex];
        this.touchList.touchStartY = firstTouch.pageY;
        this.currentElIndex = this.targetIndex
        this.scroll.scrollTo(0, -top)
        this.currentScrollIndex = this.currentElIndex;
      },
      isTouchMove(ev) {
        let firstTouch = ev.touches[0];
        this.touchList.touchMoveY = firstTouch.pageY;
        // 计算手指滑动距离
        let moveDistance = this.touchList.touchMoveY - this.touchList.touchStartY;
        // 相当于移动了多少个
        let moveIndex = Math.floor(moveDistance / this.rightItemHeight);
        // 当前应该移动到那一个首字母
        let currentInitials = moveIndex + this.currentScrollIndex;
        if (currentInitials < 0) {
          currentInitials = 0
        }
        if (currentInitials > this.$refs.rightListItem.length - 1) {
          currentInitials = this.$refs.rightListItem.length - 1;
        }
        // console.log("当前应该移动到那一个首字母:", currentInitials)

        this.currentElIndex = currentInitials;
        this.jumpSearchListItem(this.currentElIndex);
      },
      isTouchEnd(ev) {
        this.touchList.isTouch = true;
        let firstTouch = ev.touches[0];

        this.currentScrollIndex = this.currentElIndex;
      },
      _initBScroll() {
        let me = this;
        this.scroll.on('scroll', (scrollPosition) => {
          me.$set(this, 'pageY', scrollPosition.y)
        })
      },
    },
    watch: {
      searchList(newVal) {
        let self = this;
        let copyData = JSON.parse(JSON.stringify(newVal));
        this.$set(this, "searchListCopy", copyData);
        for (let key in this.searchList) {
          for (let i = 0; i < this.searchList[key].length; i++) {
            this.cityArr.push(this.searchList[key][i]);
          }
        };
        setTimeout(() => {
          let searchListItems = this.$refs.searchListItem;
          self.rightItemHeight = this.$refs.rightListItem[0].clientHeight;
          let baseScroll = 0;
          self.heightArray.push(baseScroll);
          for (let i = 0; i < searchListItems.length; i++) {
            let elHeight = searchListItems[i].getBoundingClientRect().height;
            baseScroll += elHeight;
            self.heightArray.push(baseScroll);
          }

          this.scroll = new BScroll(this.$refs.searchListWrapper, {
            probeType: 3,
            click: true
          });
          this._initBScroll();
        }, 20)
      },
      pageY(newY) {
        let top = -newY;
        let searchListItem = this.$refs.searchListItem;
        for (let i = 0; i < searchListItem.length; i++) {
          if (i <= (searchListItem.length - 1) && top >= this.heightArray[i] && top < this.heightArray[(i + 1)]) {
            this.currentElIndex = i;
          }
        }
      }
    }
  }
</script>

<style scoped lang="scss">
  .search-list {
    position: relative;
    overflow: hidden;
    .search {
      position: relative;
      z-index: 99;
      background: #fff;
      .search-btn {

      }
      .search-input {
        border: 1px solid black;
      }
    }
    .search-list-wrapper {
      position: relative;
      z-index: 10;
      height: 667-24px;
      // overflow-y: auto;
      /*background: paleturquoise;*/
      .search-list {
        .list-item {
          padding-bottom: 300px;
          &:nth-last-of-type(1) {
            padding-bottom: 600px;
          }
        }
      }
    }
    /*右侧首字母列表*/
    .right-list-wrapper {
      position: absolute;
      right: 0;
      top: 50%;
      transform: translateY(-50%);
      z-index: 100;
      .right-list {
        .list-item {
          font-size: 20px;
          line-height: 24px;
          /*padding:0 5px;*/

        }
        .current {
          color: red;
        }
      }
    }
  }

</style>
