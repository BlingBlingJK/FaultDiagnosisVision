<template>
  <div class="screen-container" :style="containerStyle">
    <header class="screen-header">
      <div>
        <img :src="headerSrc" alt="" />
      </div>

      <span class="title">高速列车故障诊断数据可视化平台</span>
      <div class="title-right">
        <img :src="themeSrc" class="qiehuan" @click="handleChangeTheme" />
        <span class="datetime"
          >{{
            dateyear +
            "-" +
            datemonth +
            "-" +
            dateday +
            " " +
            datetime +
            ":" +
            datamous +
            ":" +
            getSeconds
          }}
        </span>
      </div>
    </header>
    <div class="screen-body">
      <section class="screen-left">
        <div
          id="left-bottom"
          :class="[fullScreenStatus.seller ? 'fullscreen' : '']"
        >
          <!-- 商家销售金额图表 -->
          <SellerView ref="seller"></SellerView>
          <div class="resize">
            <!-- icon-compress-alt -->
            <span
              @click="changeSize('seller')"
              :class="[
                'iconfont',
                fullScreenStatus.seller
                  ? 'icon-compress-alt'
                  : 'icon-expand-alt',
              ]"
            ></span>
          </div>
        </div>
        <div
          id="left-top"
          :class="[fullScreenStatus.trend ? 'fullscreen' : '']"
        >
          <!-- 销量趋势图表 -->
          <TrendView ref="trend"></TrendView>
          <div class="resize">
            <!-- icon-compress-alt -->
            <span
              @click="changeSize('trend')"
              :class="[
                'iconfont',
                fullScreenStatus.trend
                  ? 'icon-compress-alt'
                  : 'icon-expand-alt',
              ]"
            ></span>
          </div>
        </div>
      </section>
      <section class="screen-middle">
        <div
          id="middle-top"
          :class="[fullScreenStatus.map ? 'fullscreen' : '']"
        >
          <!-- 商家分布图表 -->
          <MapView ref="map"></MapView>
          <div class="resize">
            <!-- icon-compress-alt -->
            <span
              @click="changeSize('map')"
              :class="[
                'iconfont',
                fullScreenStatus.map ? 'icon-compress-alt' : 'icon-expand-alt',
              ]"
            ></span>
          </div>
        </div>
        <div
          id="middle-bottom"
          :class="[fullScreenStatus.rank ? 'fullscreen' : '']"
        >
          <!-- 地区销量排行图表 -->
          <RankView ref="rank"></RankView>
          <div class="resize">
            <!-- icon-compress-alt -->
            <span
              @click="changeSize('rank')"
              :class="[
                'iconfont',
                fullScreenStatus.rank ? 'icon-compress-alt' : 'icon-expand-alt',
              ]"
            ></span>
          </div>
        </div>
      </section>
      <section class="screen-right">
        <div id="right-top" :class="[fullScreenStatus.hot ? 'fullscreen' : '']">
          <!-- 热销商品占比图表 -->
          <HotView ref="hot"></HotView>
          <div class="resize">
            <!-- icon-compress-alt -->
            <span
              @click="changeSize('hot')"
              :class="[
                'iconfont',
                fullScreenStatus.hot ? 'icon-compress-alt' : 'icon-expand-alt',
              ]"
            ></span>
          </div>
        </div>
        <div
          id="right-bottom"
          :class="[fullScreenStatus.stock ? 'fullscreen' : '']"
        >
          <!-- 库存销量分析图表 -->
          <StockView ref="stock"></StockView>
          <div class="resize">
            <!-- icon-compress-alt -->
            <span
              @click="changeSize('stock')"
              :class="[
                'iconfont',
                fullScreenStatus.stock
                  ? 'icon-compress-alt'
                  : 'icon-expand-alt',
              ]"
            ></span>
          </div>
        </div>
      </section>
    </div>
  </div>
</template>

<script>
import HotView from "@/components/HotView.vue";
import MapView from "@/components/MapView.vue";
import RankView from "@/components/RankView.vue";
import SellerView from "@/components/SellerView.vue";
import StockView from "@/components/StockView.vue";
import TrendView from "@/components/TrendView.vue";
import { mapState } from "vuex";
import { getThemeValue } from "@/utils/theme_utils";
export default {
  created() {
    // 注册接收到数据的回调函数
    this.$socket.registerCallBack("fullScreen", this.recvData);
    this.$socket.registerCallBack("themeChange", this.recvThemeChange);
  },
  destroyed() {
    this.$socket.unRegisterCallBack("fullScreen");
    this.$socket.unRegisterCallBack("themeChange");
  },
  data() {
    return {
      // 定义每一个图表的全屏状态
      timer: "",
      dateyear: new Date().getFullYear() + "",
      datemonth: new Date().getMonth() + 1 + "",
      dateday: new Date().getDate() + "",

      datetime: new Date().getHours() + "",
      datamous: new Date().getMinutes() + "",
      getSeconds: new Date().getSeconds() + "",
      fullScreenStatus: {
        trend: false,
        seller: false,
        map: false,
        rank: false,
        hot: false,
        stock: false,
      },
    };
  },
  mounted() {
    this.timer = setInterval(() => {
      this.datetime =
        new Date().getHours() < 10
          ? "0" + new Date().getHours()
          : new Date().getHours() + "";
      this.datamous =
        new Date().getMinutes() < 10
          ? "0" + new Date().getMinutes()
          : new Date().getMinutes() + "";
      this.getSeconds =
        new Date().getSeconds() < 10
          ? "0" + new Date().getSeconds()
          : new Date().getSeconds() + "";
    }, 1000);
  },
  beforDestroy() {
    if (this.timer) {
      clearInterval(this.timer);
    }
  },

  methods: {
    changeSize(chartName) {
      // 1.改变fullScreenStatus的数据
      // this.fullScreenStatus[chartName] = !this.fullScreenStatus[chartName]
      // 2.需要调用每一个图表组件的screenAdapter的方法
      // this.$refs[chartName].screenAdapter()
      // this.$nextTick(() => {
      //   this.$refs[chartName].screenAdapter()
      // })
      // 将数据发送给服务端
      const targetValue = !this.fullScreenStatus[chartName];
      this.$socket.send({
        action: "fullScreen",
        socketType: "fullScreen",
        chartName: chartName,
        value: targetValue,
      });
    },
    // 接收到全屏数据之后的处理
    recvData(data) {
      // 取出是哪一个图表需要进行切换
      const chartName = data.chartName;
      // 取出, 切换成什么状态
      const targetValue = data.value;
      this.fullScreenStatus[chartName] = targetValue;
      this.$nextTick(() => {
        this.$refs[chartName].screenAdapter();
      });
    },
    handleChangeTheme() {
      // 修改VueX中数据
      // this.$store.commit('changeTheme')
      this.$socket.send({
        action: "themeChange",
        socketType: "themeChange",
        chartName: "",
        value: "",
      });
    },
    recvThemeChange() {
      this.$store.commit("changeTheme");
    },
  },
  components: {
    HotView,
    MapView,
    RankView,
    SellerView,
    StockView,
    TrendView,
  },
  computed: {
    headerSrc() {
      return "/static/img/" + getThemeValue(this.theme).headerBorderSrc;
    },
    themeSrc() {
      return "/static/img/" + getThemeValue(this.theme).themeSrc;
    },
    containerStyle() {
      return {
        backgroundColor: getThemeValue(this.theme).backgroundColor,
        color: getThemeValue(this.theme).titleColor,
      };
    },
    ...mapState(["theme"]),
  },
};
</script>
<style lang="less" scoped>
// 全屏样式的定义
.fullscreen {
  position: fixed !important;
  top: 0 !important;
  left: 0 !important;
  width: 100% !important;
  height: 100% !important;
  margin: 0 !important;
  z-index: 100;
}

.screen-container {
  width: 100%;
  height: 100%;
  padding: 0 20px;
  background-color: #161522;
  color: #fff;
  box-sizing: border-box;
}
.screen-header {
  width: 100%;
  height: 64px;
  font-size: 20px;
  position: relative;
  > div {
    img {
      width: 100%;
    }
  }
  .title {
    position: absolute;
    left: 50%;
    top: 50%;
    font-size: 20px;
    transform: translate(-50%, -50%);
  }
  .title-right {
    display: flex;
    align-items: center;
    position: absolute;
    right: 0px;
    top: 50%;
    transform: translateY(-80%);
  }
  .qiehuan {
    width: 28px;
    height: 21px;
    cursor: pointer;
  }
  .datetime {
    font-size: 15px;
    margin-left: 10px;
  }
  .logo {
    position: absolute;
    left: 0px;
    top: 50%;
    transform: translateY(-80%);
    img {
      height: 35px;
      width: 128px;
    }
  }
}
.screen-body {
  width: 100%;
  height: 100%;
  display: flex;
  margin-top: 10px;
  .screen-left {
    height: 100%;
    width: 27.6%;
    #left-top {
      height: 45%;
      margin-top: 25px;
      position: relative;
    }
    #left-bottom {
      height: 39%;
      position: relative;
    }
  }
  .screen-middle {
    height: 100%;
    width: 41.5%;
    margin-left: 1.6%;
    margin-right: 1.6%;
    #middle-top {
      width: 100%;
      height: 56%;
      position: relative;
    }
    #middle-bottom {
      margin-top: 25px;
      width: 100%;
      height: 28%;
      position: relative;
    }
  }
  .screen-right {
    height: 100%;
    width: 27.6%;
    #right-top {
      height: 46%;
      position: relative;
    }
    #right-bottom {
      height: 38%;
      margin-top: 25px;
      position: relative;
    }
  }
}
.resize {
  position: absolute;
  right: 20px;
  top: 20px;
  cursor: pointer;
}
</style>
