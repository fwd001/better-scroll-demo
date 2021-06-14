<template>
  <div id="scroll-demo">
    <div class="scroll_wrapper">
      <!-- 菜单区域 -->
      <div class="menu_wrapper" ref="menu_wrapper">
        <ul class="menu_list" ref="menu_list">
          <li
            class="menu_item"
            @click="checkMenu(index)"
            :class="{ on: index === currentIndex }"
            v-for="(item, index) in list"
            :key="item.key"
            style="border-bottom: 1px solid green"
          >
            <div style="padding-bottom: 30px">{{ item.name }}</div>
            <div :class="['menu_item_img', { on_img: index === currentIndex }]">
              <img :src="item.image" />
            </div>
          </li>
        </ul>
      </div>
      <!-- 内容区域 -->
      <div class="content_wrapper" ref="contents_wrapper">
        <ul class="contents_list" ref="contents_list">
          <li
            class="content_item"
            v-for="(item, index) in list"
            :key="item.key"
          >
            <h3 class="title">{{ item.name }}</h3>
            <ul class="content_list">
              <img class="content_list_img" :src="item.image" />
            </ul>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import Bscroll from "better-scroll";

// 本地非响应式数据
let localData = {
  // 菜单列表内容长度
  menuListHeight: 0,
  // 菜单列表盒子长度
  menuWrapperHeight: 0,
  // 内容列表长度
  contentListHeight: 0,
  // 内容盒子长度
  contentWrapperHeight: 0,
  // 点击触发
  clickEmit: false,
};

export default {
  data() {
    return {
      list: [
        {
          key: "key0",
          name: "第0章",
          content: "第0章内容",
          image: "/pdf/pdf_00.png",
        },
        {
          key: "key1",
          name: "第1章",
          content: "第1章内容",
          image: "/pdf/pdf_01.png",
        },
        {
          key: "key2",
          name: "第2章",
          content: "第2章内容",
          image: "/pdf/pdf_02.png",
        },
        {
          key: "key3",
          name: "第3章",
          content: "第3章内容",
          image: "/pdf/pdf_03.png",
        },
        {
          key: "key4",
          name: "第4章",
          content: "第4章内容",
          image: "/pdf/pdf_04.png",
        },
        {
          key: "key5",
          name: "第5章",
          content: "第5章内容",
          image: "/pdf/pdf_05.png",
        },
        {
          key: "key6",
          name: "第6章",
          content: "第6章内容",
          image: "/pdf/pdf_06.png",
        },
        {
          key: "key7",
          name: "第7章",
          content: "第7章内容",
          image: "/pdf/pdf_07.png",
        },
        {
          key: "key8",
          name: "第8章",
          content: "第8章内容",
          image: "/pdf/pdf_08.png",
        },
        {
          key: "key9",
          name: "第9章",
          content: "第9章内容",
          image: "/pdf/pdf_09.png",
        },
      ],
      tops: [],
      scrollY: 0,
      currentIndex: 0,
    };
  },
  mounted() {
    // 2 进行初始化滚动
    this.initScroll();
    // 3 收集左右侧列表各个分类的高度
    this.initTops();
  },
  computed: {},
  watch: {},
  methods: {
    initScroll() {
      // 初始化左边菜单滚动
      this.menuScroll = new Bscroll(".menu_wrapper", {
        click: true,
        probeType: 3,
        mouseWheel: true,
        bounce: false,
      });
      // 初始化右边内容滚动
      this.contentScroll = new Bscroll(".content_wrapper", {
        click: true,
        // 1:只有滚动的时候会触发scroll事件 会截流,
        // 2:只有滚动的时候会实时的触发scroll事件 不会节流,
        // 3 滚动的时候会实时触发scroll事件,惯性滑动的时候也会触发scroll事件
        probeType: 3,
        mouseWheel: true,
        bounce: false,
      });
      this.menuScroll.on("scroll", ({ x, y }) => {
        const scrollY = Math.abs(y);
        this.scrollY = scrollY;
      });
      // 5.1 监听内容列表的滚动,得到实时滚动的位置
      this.contentScroll.on("scroll", ({ x, y }) => {
        if (localData.clickEmit) return;
        const scrollY = Math.abs(y);
        this.calcIndex(scrollY);
        const my =
          (scrollY * localData.menuListHeight) / localData.contentListHeight;
        if (localData.menuListHeight - my > localData.menuWrapperHeight) {
          this.menuScroll.scrollTo(0, -my, 300);
        }
      });

      this.contentScroll.on("scrollEnd", ({ x, y }) => {
        const scrollY = Math.abs(y);
        localData.clickEmit = false;
      });
    },
    initTops() {
      let cl = this.$refs.contents_list;
      let lis = cl.children;
      let ml = this.$refs.menu_list;
      let mlis = ml.children;
      let mw = this.$refs.menu_wrapper;
      let cw = this.$refs.contents_wrapper;
      let mtop = 0;
      let ctop = 0;
      let tops = [];
      tops.push({
        m: mtop,
        c: ctop,
      });
      localData.menuListHeight = ml.clientHeight;
      localData.menuWrapperHeight = mw.clientHeight;
      localData.contentListHeight = cl.clientHeight;
      localData.contentWrapperHeight = cw.clientHeight;
      // 将list分类到顶部的高度
      Array.prototype.slice.call(lis).forEach((li, index) => {
        mtop += mlis[index].clientHeight;
        ctop += li.clientHeight;
        tops.push({
          m: mtop,
          c: ctop,
        });
      });
      this.tops = tops;
    },
    // 计算index
    calcIndex(scrollY) {
      const { tops, list } = this;
      let index = 0;
      // 5.3 找到分类的下标索引,实现联动左侧列表
      index = tops.findIndex((top, i) => {
        return (
          scrollY >= top.c - localData.contentWrapperHeight / 2 &&
          scrollY < tops[i + 1].c - localData.contentWrapperHeight / 2
        );
      });
      console.log("calcIndex", scrollY, index, localData.contentWrapperHeight);

      const bottomScrollY =
        localData.contentListHeight - localData.contentWrapperHeight;
      if (bottomScrollY <= scrollY) {
        index = list.length - 1;
      }

      this.currentIndex = index;
    },

    // 4 实现左边菜单点击联动右列表
    checkMenu(index) {
      if (this.currentIndex === index) return;
      localData.clickEmit = true;
      this.currentIndex = index;
      const { tops } = this;
      const scrollY = tops[index].c;
      /**
       * scrollTo
       * 是better-scroll 模块的方法 用于滑动到指定位置
       * 第一个参数:横轴方向(也就是坐标轴的x轴方向)滑动的大小
       * 第二个参数:纵轴方向(y轴方向)滑动的大小
       */
      // y 轴方向滑动的大小为对应食物分类收集的高度的负值
      const bottomScrollY =
        localData.contentListHeight - localData.contentWrapperHeight;
      if (bottomScrollY > scrollY) {
        this.contentScroll.scrollTo(0, -scrollY, 300);
      } else {
        this.contentScroll.scrollTo(0, -bottomScrollY, 300);
      }
    },
  },
};
</script>

<style lang="less">
html,
body {
  margin: 0;
  padding: 0;
}
h3 {
  margin: 0;
  padding: 0;
}
ul,
li {
  list-style: none;
  padding: 0;
  margin: 0;
}
#scroll-demo {
  height: 100vh;
  width: 100vw;
}
img {
  height: 100%;
}
.scroll_wrapper {
  background-color: #ccc;
  display: flex;
  height: 100%;
  overflow: hidden;
  .menu_wrapper {
    width: 140px;
    .menu_list {
      .menu_item {
        width: 100px;
        height: 150px;
        text-align: center;
        display: flex;
        align-items: center;
        flex-direction: column;
        padding: 20px;
        .menu_item_img {
          overflow: hidden;
          &.on_img {
            border: 2px solid red;
          }
        }
        // &.on {
        //   background-color: red;
        // }
      }
    }
  }
  .content_wrapper {
    flex: 1;
    .contents_list {
      .content_item {
        .title {
          font-size: 16px;
          background: #f1f3f4;
          padding: 5px 0;
          padding-left: 15px;
        }
        .content_list {
          height: 700px;
          background-color: #eee;
          display: flex;
          flex-direction: column;
          align-items: center;
          padding: 20px;
          .content_list_img {
            height: 100%;
          }
        }
      }
    }
  }
}
</style>
