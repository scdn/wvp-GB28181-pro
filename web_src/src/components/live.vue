<template>
  <div id="devicePosition" style="width:100vw; height: 91vh">
    <el-container v-loading="loading" style="height: 91vh;" element-loading-text="拼命加载中">
      <el-aside width="300px" style="background-color: #ffffff">
        <DeviceTree :clickEvent="clickEvent" :contextMenuEvent="contextMenuEvent"></DeviceTree>
      </el-aside>
      <el-container>
        <el-header height="5vh" style="text-align: left;font-size: 17px;line-height:5vh">
          分屏:
          <i class="el-icon-full-screen btn" :class="{active:spilt==1}" @click="spilt=1"/>
          <i class="el-icon-menu btn" :class="{active:spilt==4}" @click="spilt=4"/>
          <i class="el-icon-s-grid btn" :class="{active:spilt==9}" @click="spilt=9"/>
          <i class="i-ep-more-filled btn" :class="{active:spilt==16}" @click="spilt=16">16</i>
        </el-header>
        <el-main style="padding: 0;">
          <div style="width: 99%;height: 85vh;display: flex;flex-wrap: wrap;background-color: #000;">
            <div v-for="i in spilt" :key="i" class="play-box"
                 :style="liveStyle" :class="{redborder:playerIdx == (i-1)}"
                 @click="playerIdx = (i-1)">
              <div v-if="!videoUrl[i-1]" style="color: #ffffff;font-size: 30px;font-weight: bold;">{{ i }}</div>
              <player ref="player" v-else :videoUrl="videoUrl[i-1]" fluent autoplay @screenshot="shot"
                      @destroy="destroy"/>
            </div>
          </div>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script>
import uiHeader from "../layout/UiHeader.vue";
import player from './common/jessibuca.vue'
import DeviceTree from './common/DeviceTree.vue'

export default {
  name: "live",
  components: {
    uiHeader, player, DeviceTree
  },
  data() {
    return {
      videoUrl: [''],
      spilt: 1,//分屏
      playerIdx: 0,//激活播放器

      updateLooper: 0, //数据刷新轮训标志
      count: 15,
      total: 0,

      //channel
      loading: false
    };
  },
  mounted() {

  },
  created() {
    this.checkPlayByParam()
  },

  computed: {
    liveStyle() {
      let style = {width: '100%', height: '100%'}
      switch (this.spilt) {
        case 4:
          style = {width: '49%', height: '49%'}
          break
        case 9:
          style = {width: '32%', height: '32%'}
          break
        case 16:
          style = {width: '24.5%', height: '24%'}
          break
      }
      this.$nextTick(() => {
        for (let i = 0; i < this.spilt; i++) {
          const player = this.$refs.player
          player && player[i] && player[i].updatePlayerDomSize()
        }
      })
      return style
    }
  },
  watch: {
    spilt(newValue) {
      console.log("切换画幅;" + newValue)
      let that = this
      for (let i = 1; i <= newValue; i++) {
        if (!that.$refs['player' + i]) {
          continue
        }
        this.$nextTick(() => {
          if (that.$refs['player' + i] instanceof Array) {
            that.$refs['player' + i][0].resize()
          } else {
            that.$refs['player' + i].resize()
          }
        })

      }
      window.localStorage.setItem('split', newValue)
    },
    '$route.fullPath': 'checkPlayByParam'
  },
  destroyed() {
    clearTimeout(this.updateLooper);
  },
  methods: {
    destroy(idx) {
      console.log(idx);
      this.clear(idx.substring(idx.length - 1))
    },
    clickEvent: function (device, data, isCatalog) {
      if (data.channelId && !isCatalog) {
        if (device.online === 0) {
          this.$message.error('设备离线!不允许点播');
        }else {
          this.sendDevicePush(data)
        }
      }
    },
    contextMenuEvent: function (device, event, data, isCatalog) {

    },
    //通知设备上传媒体流
    sendDevicePush: function (itemData) {
      // if (itemData.status === 0) {
      //   this.$message.error('设备离线!');
      //   return
      // }
      this.save(itemData)
      let deviceId = itemData.deviceId;
      // this.isLoging = true;
      let channelId = itemData.channelId;
      console.log("通知设备推流1：" + deviceId + " : " + channelId);
      let idxTmp = this.playerIdx
      let that = this;
      this.loading = true
      this.$axios({
        method: 'get',
        url: '/api/play/start/' + deviceId + '/' + channelId
      }).then(function (res) {
        if (res.data.code === 0 && res.data.data) {
          let videoUrl;
          if (location.protocol === "https:") {
            videoUrl = res.data.data.wss_flv;
          } else {
            videoUrl = res.data.data.ws_flv;
          }
          itemData.playUrl = videoUrl;
          that.setPlayUrl(videoUrl, idxTmp);
        } else {
          that.$message.error(res.data.msg);
        }
      }).catch(function (e) {
      }).finally(() => {
        that.loading = false
      });
    },
    setPlayUrl(url, idx) {
      this.$set(this.videoUrl, idx, url)
      let _this = this
      setTimeout(() => {
        window.localStorage.setItem('videoUrl', JSON.stringify(_this.videoUrl))
      }, 100)

    },
    checkPlayByParam() {
      let {deviceId, channelId} = this.$route.query
      if (deviceId && channelId) {
        this.sendDevicePush({deviceId, channelId})
      }
    },
    shot(e) {
      // console.log(e)
      // send({code:'image',data:e})
      var base64ToBlob = function (code) {
        let parts = code.split(';base64,');
        let contentType = parts[0].split(':')[1];
        let raw = window.atob(parts[1]);
        let rawLength = raw.length;
        let uInt8Array = new Uint8Array(rawLength);
        for (let i = 0; i < rawLength; ++i) {
          uInt8Array[i] = raw.charCodeAt(i);
        }
        return new Blob([uInt8Array], {
          type: contentType
        });
      };
      let aLink = document.createElement('a');
      let blob = base64ToBlob(e); //new Blob([content]);
      let evt = document.createEvent("HTMLEvents");
      evt.initEvent("click", true, true); //initEvent 不加后两个参数在FF下会报错  事件类型，是否冒泡，是否阻止浏览器的默认行为
      aLink.download = '截图';
      aLink.href = URL.createObjectURL(blob);
      aLink.click();
    },
    save(item) {
      let dataStr = window.localStorage.getItem('playData') || '[]'
      let data = JSON.parse(dataStr);
      data[this.playerIdx] = item
      window.localStorage.setItem('playData', JSON.stringify(data))
    },
    clear(idx) {
      let dataStr = window.localStorage.getItem('playData') || '[]'
      let data = JSON.parse(dataStr);
      data[idx - 1] = null;
      console.log(data);
      window.localStorage.setItem('playData', JSON.stringify(data))
    },
  }
};
</script>
<style>
.i-ep-more-filled {
  --un-icon: url("data:image/svg+xml;utf8,%3Csvg viewBox='0 0 1024 1024' width='0.6em' height='0.6em' xmlns='http://www.w3.org/2000/svg' %3E%3Cpath fill='currentColor' d='M336 112h144v160h-144zM544 336h144v144h-144zM112 336h160v144H112zM272 112H120c-4.8 0-8 3.2-8 8V272h160V112zM544 544h144v144h-144zM544 112h144v160h-144zM336 544h144v144h-144zM336 336h144v144h-144zM752 336h160v144H752zM112 544h160v144H112zM904 112H752v160h160V120c0-4.8-3.2-8-8-8zM752 912h152c4.8 0 8-3.2 8-8V752H752v160zM752 544h160v144H752zM112 752v152c0 4.8 3.2 8 8 8H272V752H112zM544 752h144v160h-144zM336 752h144v160h-144z'/%3E%3C/svg%3E");
  -webkit-mask: var(--un-icon) no-repeat;
  mask: var(--un-icon) no-repeat;
  -webkit-mask-size: 100% 100%;
  mask-size: 100% 100%;
  background-color: currentColor;
  color: inherit;
  width: 0.8em;
  height: 0.8em;
}
.btn {
  margin: 0 10px;

}

.btn:hover {
  color: #409EFF;
}

.btn.active {
  color: #409EFF;

}

.redborder {
  border: 2px solid red !important;
}

.play-box {
  background-color: #000000;
  border: 2px solid #505050;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
<style>
.videoList {
  display: flex;
  flex-wrap: wrap;
  align-content: flex-start;
}

.video-item {
  position: relative;
  width: 15rem;
  height: 10rem;
  margin-right: 1rem;
  background-color: #000000;
}

.video-item-img {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
  width: 100%;
  height: 100%;
}

.video-item-img:after {
  content: "";
  display: inline-block;
  position: absolute;
  z-index: 2;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
  width: 3rem;
  height: 3rem;
  background-image: url("../assets/loading.png");
  background-size: cover;
  background-color: #000000;
}

.video-item-title {
  position: absolute;
  bottom: 0;
  color: #000000;
  background-color: #ffffff;
  line-height: 1.5rem;
  padding: 0.3rem;
  width: 14.4rem;
}

.baidumap {
  width: 100%;
  height: 100%;
  border: none;
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  margin: auto;
}

/* 去除百度地图版权那行字 和 百度logo */
.baidumap > .BMap_cpyCtrl {
  display: none !important;
}

.baidumap > .anchorBL {
  display: none !important;
}
</style>
