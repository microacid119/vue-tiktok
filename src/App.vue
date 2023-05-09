<template lang="pug">
.swiper-container(ref='swiperRef')
  .swiper-wrapper
    .swiper-slide(v-for="(item, i) in videoData")
      video.player(:poster="item.cover" preload='none' muted='' loop autoplay @click="toggleVideo")
      .category
        h2(@click="switchCategory()") Following
        h2(@click="switchCategory()") For you
      img.icon-pause(v-show="!isPlayVideo" src="./assets/play.png" @click="toggleVideo")
      .img-play
        .progress-bar
          .bar
          input.progress-bar__range(type='range' min='0' max='100' value='0' step='0.1')

</template>

<script setup>
import Swiper from 'swiper'
import 'swiper/swiper-bundle.css'
import { ref, onMounted, onUnmounted, toRaw, watch } from 'vue'
const isPlayVideo = ref(true)
const videoData = ref([])
const swiperRef = ref(null)
const swiperActiveIndex = ref(0)
const currtentCategory = ref('following')
let mySwiper = null

const toggleVideo = () => {
  const videos = document.querySelectorAll('video')
  if (videos[mySwiper.activeIndex].paused) {
    videos[mySwiper.activeIndex].play()
    isPlayVideo.value = true
  } else {
    videos[mySwiper.activeIndex].pause()
    isPlayVideo.value = false
  }
}

onMounted(() => {
  getFollowingList()
  setupSwiper()
  setupVideosAndProgressbar()
})

watch(() => swiperActiveIndex.value, (newValue, oldValue) => {
    const videos = document.querySelectorAll('.player')
    isPlayVideo.value = true
    videos[oldValue].pause()
    videos[newValue].play()
  })
  
onUnmounted(() => {
  mySwiper.destroy()
})

const getFollowingList = async() => {
  const response = await fetch('http://localhost:3000/following_list')
  const data = await response.json()
  videoData.value = data.items
  // console.log('videoData', toRaw(videoData.value))
}

const getForyouList = async() => {
  const response = await fetch('http://localhost:3000/for_you_list')
  const data = await response.json()
  videoData.value = data.items
  // console.log('videoData', toRaw(videoData.value))
}

function setupSwiper() {
  mySwiper = new Swiper(swiperRef.value, {
    // swiper 的配置项
    direction: 'vertical',
    on: {
      slideChangeTransitionEnd() {
        swiperActiveIndex.value = this.activeIndex
      }
    }
  })
}

function setupVideosAndProgressbar (){
    // 定义计时器变量
    let timer = null;
  
    // 开始计时器
    timer = setInterval(() => {
      const videos = document.querySelectorAll('.player')
  
      // 判断条件是否达成
      if (videos.length > 0) {
        // 达成条件，停止计时器
        // setup videos
        for (let i = 0; i < videoData.value.length; i++) {
          if (Hls.isSupported()) {
            const hls = new Hls()
            hls.loadSource(toRaw(videoData.value[i].play_url))
            hls.attachMedia(videos[i])
            if (mySwiper.activeIndex === i) {
              hls.on(Hls.Events.MANIFEST_PARSED,function() {
                videos[i].play()
              })
            }
          }
          Plyr.setup(videos[i])
  
          // setup progress bar
          const progressbarInput = document.querySelectorAll('.progress-bar__range')
          const bar = document.querySelectorAll('.bar')
  
          // 监听 range 元素的 input 事件
          progressbarInput[i].addEventListener('input', function() {
            // 将视频播放进度设置为 range 元素的 value 值
            videos[i].currentTime = (videos[i].duration / 100) * this.value
          })
  
          // 监听视频元素的 timeupdate 事件
          videos[i].addEventListener('timeupdate', function() {
            // 将 range 元素的 value 值设置为当前视频播放进度的百分比
            progressbarInput[i].value = (100 / videos[i].duration) * videos[i].currentTime
            const percent = (videos[i].currentTime / videos[i].duration) * 100
            bar[i].style.width = `${percent}%`
          });
        }
        clearInterval(timer);
  
      }
    }, 100);
  }
  
  function switchCategory () {
    if (currtentCategory.value === 'following') {
      getForyouList()
      currtentCategory.value = 'foryou'
    } else {
      getFollowingList()
      currtentCategory.value = 'following'
    }
    isPlayVideo.value = true
    setupVideosAndProgressbar()
  }

</script>

<style lang="stylus">
body, div, h1, h2, h3, h4, h5, h5, ul, li, p, input, label, a
  margin 0
  padding 0
  outline none

body
  background #242424

h2
  color #fff

video
  height 100vh
  margin 0 auto

.img-play
  position absolute
  width 100%
  height 10px
  display flex
  justify-content center
  align-items center
  z-index 30
  bottom 0

.swiper-container
  width 100vw
  height 100vh

.swiper-slide
  text-align center
  font-size 18px
  background #fff
  height 100vh
  display flex
  justify-content center
  align-items center
  overflow hidden

/* 进度条样式 */
.progress-bar {
  width: 100%;
  height: 10px;
  background-color: #eee;
  position absolute
  bottom 0
}

.bar
  position absolute
  height 10px
  width 50%
  background red


/* 进度条内部元素样式 */
.progress-bar__range {
  width: 100%;
  height: 10px;
  -webkit-appearance: none;
  background-color: transparent;
  position absolute
  left 0
}

/* 拖拽圆点样式 */
.progress-bar__range::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 10px;
  height: 10px;
  border-radius: 50%;
}

/* 点击后拖拽圆点的样式 */
.progress-bar__range::-moz-range-thumb {
  width: 10px;
  height: 10px;
  border-radius: 50%;
}

.poster
  position absolute
  width 100vw
  height 100vh
  z-index 20

.icon-pause
  position absolute
  left calc(50vw - 50px)
  z-index 50
  width 100px
  height 100px

.category
  position absolute
  z-index 10
  display flex
  justify-content center
  top 2rem
  h2
    margin-right 10px
    -webkit-text-stroke: 1px #fff;
    font-family: sans; color: red;
    &:first-child
      border-right 1px solid #fff
      padding-right 10px
</style>
