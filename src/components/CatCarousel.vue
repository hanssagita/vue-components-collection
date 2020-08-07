<template>
  <div>
    <div class="cat-carousel-container">
      <div
        :class="['cat-carousel__navigation', {'cat-carousel__navigation--end': onFirstPage}]"
      >
        <template>
          <slot name="prev-navigation" :prev="prev">
            <div class="cat-carousel__default-nav cat-carousel__default-nav--left" @click="prev">
              <img src="https://www.static-src.com/siva/asset//07_2020/arrow-blue-small.png">
            </div>
          </slot>
        </template>
      </div>
      <div
        ref="carouselContent"
        class="cat-carousel__content"
      >
        <div
          ref="carouselWrapper"
          :style="wrapperStyles"
          class="cat-carousel__content__wrapper"
          @touchstart="touchStart"
          @touchmove="touchMove"
        >
          <div
            v-for="(item, index) in items"
            ref="carouselItem"
            :key="index"
            class="cat-carousel__content__wrapper__item"
            :style="carouselItemStyles"
          >
            <template>
              <slot
                name="item"
                :data="item"
                :index="index"
              />
            </template>
          </div>
        </div>
      </div>
      <div
        :class="['cat-carousel__navigation cat-carousel__navigation__next',
                 {'cat-carousel__navigation--end': onLastPage}]"
      >
        <template>
          <slot name="next-navigation" :next="next">
            <div class="cat-carousel__default-nav" @click="next">
              <img src="https://www.static-src.com/siva/asset//07_2020/arrow-blue-small.png">
            </div>
          </slot>
        </template>
      </div>
    </div>
    <div
      :class="{'hide': hideIndicators}"
      class="cat-carousel__indicators"
    >
      <div
        v-for="index in maxSlide"
        v-show="validateIndicator(index - 1)"
        :key="index"
        :class="['cat-carousel__indicators__item', validateIndicator(index - 1)]"
      />
    </div>
  </div>
</template>

<script>
  const WIDTH_PAGE = 100
  const SWIPE_THRESHOLD = 80
  const CENTER_MODE_DEFAULT_CONFIG = {
    enabled: false,
    paddingCenter: 10
  }
  const INDICATORS_DEFAULT_CONFIG = {
    hideIndicators: false,
    maxIndicator: 2
  }

  export default {
    name: 'CatCarousel',
    props: {
      items: {
        type: Array,
        default: () => [],
        required: true
      },
      itemPerPage: {
        type: Number,
        default: 5
      },
      indicatorsConfig: {
        type: Object,
        default: () => {
          return INDICATORS_DEFAULT_CONFIG
        }
      },
      centerMode: {
        type: Object,
        default: () => {
          return CENTER_MODE_DEFAULT_CONFIG
        }
      }
    },
    data () {
      return {
        wrapper: {
          translateX: 0
        },
        maxSlide: 0,
        track: 0,
        trackEnd: 0,
        trackStart: 0,
        slides: [],
        normalSlideWindow: [],
        reversedSlideWindow: [],
        touchX: null
      }
    },
    mounted () {
      this.maxSlide = Math.ceil(this.items.length / this.itemPerPage)
      this.initSlides()
    },
    watch: {
      items () {
        this.wrapper = {
          translateX: 0
        }
        this.track = 0
        this.maxSlide = Math.ceil(this.items.length / this.itemPerPage)
        this.initSlides()
      },
      track(val) {
        if (val) {
          if (val < this.trackStart) {
            this.trackStart = val
            this.trackEnd = this.trackStart + this.maxIndicator
            if (this.trackEnd > this.slides.length) {
              this.trackEnd = this.slides.length
            }
          }
          if (val > this.trackEnd) {
            this.trackEnd = val
            this.trackStart = this.trackEnd - this.maxIndicator
            if (this.trackStart < 0) {
              this.trackStart = 0
            }
          }
        }
      }
    },
    computed: {
      carouselContent () {
        return this.$refs.carouselContent
      },
      carouselItem () {
        return this.$refs.carouselItem || []
      },
      carouselWrapper () {
        return this.$refs.carouselWrapper
      },
      itemWidth () {
        return this.carouselItem && this.carouselItem.length > 0 && this.carouselItem[0].clientWidth
      },
      wrapperStyles () {
        if (this.centerMode.enabled) {
          return {transform: `translateX(calc(${this.wrapper.translateX}px + ${this.centerMode.paddingCenter}%)`}
        }

        return {transform: `translateX(${this.wrapper.translateX}px)`}
      },
      onFirstPage () {
        return this.track === 0
      },
      onLastPage () {
        return this.track === this.maxSlide - 1
      },
      carouselItemStyles () {
        let width = WIDTH_PAGE / this.itemPerPage

        if (this.centerMode.enabled) {
          width = width * (1 - this.centerMode.paddingCenter / 100 * 2)
        }

        return {
          flex: `0 0 ${width}%`,
          width: `${width}%`
        }
      },
      hideIndicators () {
        return this.indicatorsConfig.hideIndicators || INDICATORS_DEFAULT_CONFIG.hideIndicators
      },
      maxIndicator () {
        return this.indicatorsConfig.maxIndicator - 1 || INDICATORS_DEFAULT_CONFIG.maxIndicator - 1
      }
    },
    methods: {
      initSlides () {
        this.trackEnd = this.maxIndicator
        this.slides = this.addSlides(this.items.length)
        this.initialSlides = this.slides
        this.reversedSlides = this.slides.slice().reverse()
      },
      validateIndicator(index) {
        if (index === this.track) {
          return 'active-indicator'
        }
        if (index >= this.trackStart && index <= this.trackEnd) {
          return 'std-indicator'
        }
        if (index === this.trackStart - 1 || index === this.trackEnd + 1) {
          return 'small-indicator'
        }
        if (index === this.trackStart - 2 || index === this.trackEnd + 2) {
          return 'micro-indicator'
        }
        return false
      },
      addSlides (itemsLength) {
        if (itemsLength <= 0) return []
        const count = Math.min(itemsLength, this.itemPerPage)
        let slides = []
        slides = slides.concat([count], this.addSlides(itemsLength-count))
        return slides
      },
      prev () {
        if (this.onFirstPage) return
        this.track--
        this.wrapper = Object.assign({}, this.wrapper,{
          translateX: this.wrapper.translateX + this.slides[this.track] * this.itemWidth
        })
        if (this.onFirstPage) this.slides = this.initialSlides
      },
      next () {
        if (this.onLastPage) return
        this.track++
        this.wrapper = Object.assign({}, this.wrapper, {
          translateX: this.wrapper.translateX - this.slides[this.track] * this.itemWidth
        })
        if (this.onLastPage) this.slides = this.reversedSlides
      },
      selectedIndicator (index) {
        return index === this.track + 1
      },
      touchStart (event) {
        this.touchX = event.touches[0].clientX
      },
      touchMove (event) {
        if (!this.touchX) return
        let currentX = event.touches[0].clientX
        let diffX = currentX - this.touchX
        if (diffX > SWIPE_THRESHOLD) {
          this.prev()
          this.touchX = null
        }
        if (diffX < -SWIPE_THRESHOLD) {
          this.next()
          this.touchX = null
        }
      }
    }
  }

</script>

<style lang="scss" scoped>
  .cat-carousel-container {
    display: flex;
    position: relative;
    width: 100%;
    height: 100%;
    margin: 0 auto;
  }
  .cat-carousel {
    &__content {
      width: 100%;
      overflow: hidden;
      display: inline-block;
      &__wrapper {
        transition: transform 0.5s ease-out;
        display: flex;
        align-items: flex-start;
        height: 100%;
        &__item {
          box-sizing: border-box;
          padding: 0 1%;
          display: flex;
          height: 100%;
        }
      }
    }
    &__navigation {
      display: flex;
      align-self: center;
      position: absolute;
      z-index: 2;
      &__next {
        right: 0;
      }
      &--end {
        display: none;
      }
    }
    &__default-nav {
      width: 48px;
      height: 48px;
      opacity: 0.88;
      box-shadow: 0 0 24px -4px rgba(0, 0, 0, 0.24);
      background-color: #ffffff;
      display: flex;
      justify-content: center;
      align-content: center;
      align-items: center;
      border-radius: 50%;
      border-right: 1px solid #eee;
      cursor: pointer;
      z-index: 2;
      @media only screen and (max-width: 768px) {
        display: none;
      }
      img {
        width: 34px;
        height: 34px;
      }
      &--left {
        img {
          transform: rotate(180deg);
        }
      }
    }
    &__indicators {
      display: flex;
      justify-content: center;
      align-items: center;
      &__item {
        width: 7.5px;
        height: 7.5px;
        margin: 2px;
        border-radius: 50%;
        background-color: rgba(214, 214, 214, 0.5);
        overflow: hidden;
        transition: all 500ms ease-out;
        text-indent: -9999px;
        &:first-child {
          margin-left: 20px;
        }
        &:last-child {
          margin-right: 20px;
        }
      }
      .active-indicator {
        width: 12px;
        height: 12px;
        margin: 1px;
        background-color: #0095da;
      }
      .small-indicator {
        width: 6px;
        height: 6px;
        margin: 3px;
        &:first-child {
          margin-left: 10px;
        }
        &:last-child {
          margin-right: 10px;
        }
      }
      .micro-indicator {
        width: 3px;
        height: 3px;
        margin: 4px;
      }
    }
  }
  .hide {
    display: none;
  }
</style>
