<template>
  <div id="introOverlay" v-bind:class="[this.hiddenIntro ? 'hide' : '']">
    <div class="introWrap">
      <div class="skip-intro-mobile" v-on:click="emitHideIntro()">
        <span>Skip Intro</span>
        <img src="@/assets/skip-down.svg">
      </div>
      <div class="intro-section">
        <div class="sectionWrap">
          <div class="logBomax"><img src="@/assets/remax-booj-horiz.svg"></div>

          <div  class="" v-if="this.videos">
            <div class="vidtitle h2 bold" v-if="this.selectedVideo.title">{{this.selectedVideo.title}}</div>

            <div id="vidWrap">
              <div id="vimeoVid"></div>
              <div class="vidDescription" v-if="this.selectedVideo.description">
                <prismic-rich-text class="pre-tooltip-text" v-if="this.selectedVideo.description" :field="this.selectedVideo.description" />
                <div class="tooltip" v-if="this.selectedVideo.tooltip">
                  <span class="ttTitle">{{this.selectedVideo.tooltip.title}}</span>
                  <span class="ttDescription">{{this.selectedVideo.tooltip.description}}</span>
                </div>
              </div>
              <ul class="vidTooltips" v-else-if="this.selectedVideo.tooltip">
                <li v-for="(tooltipInfo, index) in this.selectedVideo.tooltip" v-bind:key="index" class="tooltip">
                  <span class="ttTitle">{{tooltipInfo.title}}</span>
                  <span class="ttDescription">{{tooltipInfo.description}}</span>
                </li>
              </ul>
            </div>

            <ul class="vidList">
              <li v-for="(videoInfo, index) in this.videos" v-bind:key="index" v-bind:class="[videoInfo.isActive ? 'active' : '']" v-on:click="setActiveVid(index)">
                <div class="thumbnail">
                  <div class="thumbGradient"></div>
                  <prismic-image v-if="videoInfo.primary.thumbnail_image" class="thumb_base" :field="videoInfo.primary.thumbnail_image"/>
                  <prismic-image v-if="videoInfo.primary.thumbnail_hover_gif" class="thumb_hover" :field="videoInfo.primary.thumbnail_hover_gif"/>
                </div>
                <div class="thumbTitle">{{videoInfo.primary.thumbnail_title}} <img class="arrow" src="@/assets/cta-arrow.svg"></div>
              </li>
            </ul>
          </div>
        </div>
        <a class="scrollFixed" v-on:click="scrollDown()">
          Scroll
          <img src="@/assets/cta-arrow.svg">
        </a>
      </div>

      <div class="intro-section" v-for="(introSlide, index) in this.slides" v-bind:key="index">
        <div class="sectionWrap section2">
          <div class="introText">
            <div class="introTitle h1">{{introSlide.primary.slide_title}}</div>
            <prismic-rich-text v-if="introSlide.primary.slide_description" class="introDescription body-text" :field="introSlide.primary.slide_description" />
          </div>
          
          <div class="introGraphic">
            <iframe src="./intro-animation/index.html"></iframe>
          </div>
        </div>
        <a class="scrollFixed" v-on:click="emitHideIntro()">
          Scroll
          <img src="@/assets/cta-arrow.svg">
        </a>
      </div>
      <img class="rectLeft" src="@/assets/intro-rect-left.svg" v-bind:class="[activeClass ? 'active' : '']">
      <img class="rectRight" src="@/assets/intro-rect-right.svg" v-bind:class="[activeClass ? 'active' : '']">
      <div class="intro-exit-mobile" v-on:click="emitHideIntro()">
        <button>See How It Works</button>
        <img src="@/assets/cta-arrow.svg">
      </div>
    </div>

    
    <div class="skipIntro"><span v-on:click="emitHideIntro()">Skip Intro</span></div>
    
  </div>
</template>

<script>
import Player from '@vimeo/player'
import { eBus } from '../main'

export default {
  data () {
    return {
      hiddenIntro: false,
      activeSlideIndex: 0,
      scrollDebounce: false,
      activeClass: false,
      baseVideoSize: {
        height: 0,
        width: 0
      },
      selectedVideo: {
        title: null,
        description: null,
        tooltip: null
      },
      player: null,
      videos: [],
      slides: []
    }
  },
  methods: {
    getContent () {
      this.$prismic.client.getSingle('intro_overlay')
        .then((document) => {
          document.data.body.forEach((slice) => {

            if(slice.slice_type === 'intro_video' || slice.slice_type === 'intro_video1'){
              slice.isActive = false
              this.videos.push(slice)
            }else if(slice.slice_type === 'intro_slide'){
              this.slides.push(slice)
            }
          })
          this.setActiveVid(0)
        })
    },
    emitHideIntro (){
      eBus.$emit('introHideUpdated', true)
      // window.scrollTo(0, 0)
      // this.player.destroy()
    },
    setActiveVid (vidId) {
      let selectedVidInfo = null;
      this.videos = this.videos.filter(function(videoInfo, index){
        videoInfo.isActive = (index === vidId) ? true : false
        if(videoInfo.isActive){
          selectedVidInfo = videoInfo
        }
        return videoInfo
      })

      this.selectedVideo.title = selectedVidInfo.primary.full_title
      if(selectedVidInfo.primary.description){
        this.selectedVideo.description = selectedVidInfo.primary.description
        this.selectedVideo.tooltip = {title: selectedVidInfo.primary.tootip_title, description: selectedVidInfo.primary.tooltip_description}
      }else{
        this.selectedVideo.description = null
        this.selectedVideo.tooltip = []
        selectedVidInfo.items.forEach((tooltipInfo) => {
          if(tooltipInfo.tooltip_title && tooltipInfo.tooltip_description){
            this.selectedVideo.tooltip.push({title:tooltipInfo.tooltip_title, description:tooltipInfo.tooltip_description})
          }
        })
      }

      if(this.player !== null){
        this.player.destroy()
      }
      this.player = new Player('vimeoVid', {
        id: selectedVidInfo.primary.video.video_id,
        autoplay: true
      })
      this.player.on('loaded', this.saveBaseVideoSize)
    },
    saveBaseVideoSize () {
      let vidIframe = document.querySelector('#vimeoVid iframe')
      this.baseVideoSize.height = parseInt(vidIframe.getAttribute('height'), 10)
      this.baseVideoSize.width = parseInt(vidIframe.getAttribute('width'), 10)

      this.setResponsiveVideoSize()
    },
    setResponsiveVideoSize () {
      let containerWidth = document.getElementById('vidWrap').offsetWidth
      let ratioMulti = containerWidth/this.baseVideoSize.width
      let newHeight = this.baseVideoSize.height*ratioMulti
      let vidIframe = document.querySelector('#vimeoVid iframe')

      vidIframe.setAttribute('height', newHeight)
    },
    scrollHandler (event) {
      let windowWidth = window.outerWidth

      if(this.scrollDebounce || this.hiddenIntro || windowWidth < 1200){
        return false
      }
      this.scrollDebounce = true

      const self = this
      let scolledUp = event.deltaY > 0 ? false : true
      let introHeight = document.getElementById('introOverlay').offsetHeight

      if(scolledUp){
        this.activeSlideIndex -= 1
        this.activeClass = false
      }else{
        this.activeSlideIndex += 1
        this.activeClass = true
      }
      // window.console.log(this.activeSlideIndex)
      this.activeSlideIndex = this.activeSlideIndex < 0 ? 0 : this.activeSlideIndex
      if(this.activeSlideIndex > this.slides.length){
        this.activeSlideIndex = this.slides.length - 1
        this.emitHideIntro()
      }

      document.querySelector('#introOverlay .introWrap').setAttribute("style", "transform: translate3d(0px, "+(-1*introHeight*this.activeSlideIndex)+"px, 0px);")

      setTimeout(function(){
          self.scrollDebounce = false;
      }, 2000)
    },
    scrollDown () {
      let introHeight = document.getElementById('introOverlay').offsetHeight
       this.activeSlideIndex += 1
       this.activeClass = true
       document.querySelector('#introOverlay .introWrap').setAttribute("style", "transform: translate3d(0px, "+(-1*introHeight*this.activeSlideIndex)+"px, 0px);")


      setTimeout(function(){
          self.scrollDebounce = false;
      }, 2000)
    }
  },
  created () {
    this.getContent()
    eBus.$on('introHideUpdated', (data) => {
      this.hiddenIntro = data
      if(this.player){
        this.player.destroy();
      }
    })



    // this.emitHideIntro()


    
    window.addEventListener('resize', this.setResponsiveVideoSize)
    window.addEventListener('wheel', this.scrollHandler)
  }
}
</script>