<template>
  <div class="header" style="-webkit-app-region: drag">
    <div class="left">
      <div class="buttons" style="-webkit-app-region: no-drag;">
        <div
          @click="onClickLogo"
          class="mac-button red"
          id="screenerExit"
        >
          <Icon
            :size="9"
            type=""
          />
        </div>
        <div
          @click="hidewindow"
          class="mac-button yellow"
        >
          <Icon
            :size="9"
            type="minus"
          />
        </div>
        <div
          @click="onClickLogo"
          class="mac-button green"
        >
          <Icon
            :size="9"
            type="home"
          />
        </div>
      </div>
      <!-- 缩起播放器 -->
      <div
        @click="onClickDown"
        class="shrink-player"
        v-if="isPlayerShow"
        style="-webkit-app-region: no-drag;"
      >
        <Icon
          :backdrop="true"
          type="down"
        />
      </div>
      <!-- 路由记录器 -->
      <div
        class="history"
        v-show="!isPlayerShow"
        style="-webkit-app-region: no-drag;"
      >
        <RoutesHistory />
      </div>
    </div>
    <div class="right" style="-webkit-app-region: no-drag;">
      <div class="search-wrap">
        <Search />
      </div>
      <Theme />
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import Theme from "@/components/theme"
import Search from "@/components/search"
import RoutesHistory from "@/components/routes-history"
import { mapState, mapMutations } from "@/store/helper/music"
import { requestFullScreen, exitFullscreen, isFullscreen } from "@/utils"
if (window.require) {
  var ipc = window.require('electron').ipcRenderer
}
export default {
  methods: {
    headerToggleIndexShow() {
        var obj = document.getElementsByClassName("layout-body")[0];
        obj.style.display= "flex";
    },
    onClickLogo() {
      this.$router.push("/discovery")
    },
    onClickDown() {
      this.setPlayerShow(false)
      this.headerToggleIndexShow()
    },
    fullscreen() {
      requestFullScreen(document.documentElement)
    },
    exitFullscreen() {
      if (isFullscreen()) {
        exitFullscreen()
      }
    },
    toggleFullscreen() {
      this.isFullscreen = !this.isFullscreen
    },
    hidewindow() {
      if (window.require) {
        ipc.send('hide');
      }
    },
    ...mapMutations(["setPlayerShow"])
  },
  computed: {
    ...mapState(["isPlayerShow"])
  },
  components: { Theme, Search, RoutesHistory }
}
</script>

<style lang="scss" scoped>
@import "~@/style/element-overwrite.scss";

.header {
  display: flex;
  justify-content: space-between;
  height: $header-height;
  background-color: var(--header-bgcolor);
  padding-right: 36px;

  @include el-input-theme(
    var(--header-input-color),
    var(--header-input-bgcolor),
    var(--header-input-placeholder-color)
  );

  /deep/.iconfont {
    color: var(--header-font-color);
  }

  .left {
    padding: 14px 14px 0 8px;
    display: flex;

    .font-weight {
      white-space: nowrap;
    }

    .buttons {
      display: flex;

      &:hover {
        .mac-button > i {
          opacity: 1;
        }
      }

      .mac-button {
        @include round(12px);
        @include flex-center;
        margin-right: 8px;
        cursor: pointer;

        &.red {
          background: #ed655a;
        }

        &.green {
          background: #72be47;
        }

        &.yellow {
          background: #e0c04c;
        }

        i {
          opacity: 0;
          transition: opacity 0.3s;
          color: $black;
          font-weight: $font-weight-bold;
          transform-origin: center center;
        }
      }
    }

    .shrink-player {
      position: relative;
      top: -6px;
      margin-left: 16px;
    }

    .history {
      margin-left: 65px;
    }

    .actions {
      margin-left: 70px;
    }
  }

  .right {
    display: flex;
    align-items: center;

    .search-wrap {
      margin-right: 16px;
    }
  }
}
</style>
