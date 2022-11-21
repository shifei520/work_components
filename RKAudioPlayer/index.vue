<template>
  <div>
    <audio
      ref="audioRef"
      controls
      :src="fileurl"
      style="display: none;"
      @timeupdate="updateProgress"
    />
    <!-- 自定义音频播放条 -->
    <div class="rk-audio-box">
      <!-- 开始暂停按钮 -->
      <svg-icon
        class-name="pause-play-btn"
        :icon-class="audioStatus !== 'pause' ? 'play-fill' : 'pause-fill'"
        @click="playAudio"
      />
      <!-- 已经播放的时间 -->
      <div
        id="audioCurTime"
        class="audio-length-current"
      >
        {{ audioStart }}
      </div>
      <!-- 进度条 -->
      <div
        id="progressBarBg"
        class="progress-bar-bg"
        @click="dragToProgress"
      >
        <div
          id="progressBar"
          class="progress-bar"
        />
      </div>
      <!-- 音频总时长 -->
      <div class="audio-length-total">
        {{ duration }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'RKAudioPlayer',
  props: {
    // 需要播放的音频路径
    fileurl: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      audioStatus: 'play', // 控制播放和暂停按钮的显示隐藏
      audioStart: '0:00', // 已经播放的时间
      duration: '0:00', // 音频的总时长
      recordAudio: null // audio元素
    };
  },
  mounted() {
    this.recordAudio = this.$refs.audioRef;
    // 当监听到audio元素可以播放后获取总的播放时长
    this.recordAudio.oncanplay = () => {
      this.duration = this.transTime(this.recordAudio.duration);
    };
  },
  methods: {
    // 进度条改变时更新用于自定义进度条的进度和时间
    updateProgress(e) {
      const value = e.target.currentTime / e.target.duration;
      if (document.getElementById('progressBar')) {
        document.getElementById('progressBar').style.width = value * 100 + '%';
        if (e.target.currentTime === e.target.duration) {
          this.audioStatus = 'pause';
        }
      } else {
        this.audioStatus = 'pause';
      }

      this.audioStart = this.transTime(this.recordAudio.currentTime);
    },
    // 拖动进度条
    dragToProgress(e) {
      let wdiv = document.getElementById('progressBarBg').clientWidth;

      let ratemin = e.offsetX / wdiv;
      let rate = ratemin * 100;
      document.getElementById('progressBar').style.width = rate + '%';
      console.log(this.recordAudio.duration * ratemin);
      this.recordAudio.currentTime = this.recordAudio.duration * ratemin;
      this.recordAudio.play();
      this.audioStatus = 'pause';
    },
    // 播放暂停控制
    playAudio() {
      if (this.recordAudio.paused) {
        this.recordAudio.play();
        this.audioStatus = 'pause';
      } else {
        this.recordAudio.pause();
        this.audioStatus = 'play';
      }
    },
    /**
     * 音频播放时间换算
     * @param {number} value - 音频当前播放时间，单位秒
     */
    transTime(time) {
      var duration = parseInt(time);
      var minute = parseInt(duration / 60);
      var sec = (duration % 60) + '';
      var isM0 = ':';
      if (minute === 0) {
        minute = '00';
      } else if (minute < 10) {
        minute = '0' + minute;
      }
      if (sec.length === 1) {
        sec = '0' + sec;
      }
      return minute + isM0 + sec;
    },
  }
};
</script>

<style lang="less" scoped>
  .rk-audio-box {
    display: flex;
    align-items: center;
    width: 200px;
    height: 24px;
    font-size: 12px;
    border: 1px solid #007cee;
    box-sizing: border-box;
    border-radius: 3px;

    .progress-bar-bg {
      position: relative;
      flex: 1;
      height: 6px;
      background-color: transparent;
      cursor: pointer;
    }

    .progress-bar {
      position: absolute;
      background: linear-gradient(90deg, #fff, #007cee);
      width: 0%;
      height: 6px;
      border-radius: 4px;

      &::after {
        content: '';
        position: absolute;
        top: 50%;
        right: 0;
        width: 10px;
        height: 10px;
        transform: translate(50%, -50%);
        background-color: #007cee;
        border-radius: 50%;
      }
    }

    .pause-play-btn {
      color: #007cee;
      cursor: pointer;
      font-size: 16px;
    }

    .audio-length-current,
    .audio-length-total {
      padding: 0 10px;
      color: #666;
    }
  }
</style>