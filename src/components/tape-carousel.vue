<template>
  <div class="carousel">
    <nav v-if="false">
      <button
          class="nav prev"
          @click="changeSlide('next')">Prev
      </button>
      <button
          class="nav next"
          @click="changeSlide('prev')">Next
      </button>
    </nav>

    <div
        ref="figure"
        class="figure"
    >
      <img
          v-for="item in items"
          :key="item.Id"
          ref="slides"
          :src="item.Image.LargeUrl"
          :class="{'carousel__slide_grabbing' : isDragging}"
          class="carousel__slide"
      >
    </div>
  </div>
</template>

<script>
let startPositionX = 0;

export default {
  name: "CarProjectTape",
  props: {
    items: {
      type: Array,
      default: () => [],
    },

    gap: {
      type: Number,
      default: 20,
    },
  },

  data() {
    return {
      currImage: 0,
      isDragging: false,
      isSliding: false,
      deltaX: 0,
      isTouch: false,
    }
  },

  computed: {
    apothem() {
      const s = parseFloat(getComputedStyle(this.$refs.slides[0]).width)
      return s / (2 * Math.tan(Math.PI / this.n));
    },

    n() {
      return this.items.length;
    },

    theta() {
      if (this.n > 0) {
        return 2 * Math.PI / this.n;
      }

      return 0;
    },
  },

  mounted() {
    this.initCarousel();

  },

  methods: {
    initCarousel() {
      this.setupCarousel();

      //add listeners
      this.$refs.slides.forEach(slide => {
        slide.addEventListener(this.isTouch ? 'touchstart' : 'mousedown', this.onDragStart);
      });

      // window.addEventListener('resize', () => {
      //   setupCarousel(n, parseFloat(getComputedStyle(images[0]).width))
      // });
    },
    changeSlide(direction) {
      if (direction !== 'next' && direction !== 'prev') {
        return;
      }

      if (direction === 'next') {
        this.currImage++;
      } else {
        this.currImage--;
      }

      this.rotateCarousel(this.currImage);
    },

    setupCarousel() {
      this.$refs.figure.style.transformOrigin = `50% 50% ${-this.apothem}px`;

      this.$refs.slides.forEach((_, i) => {
        this.$refs.slides[i].style.padding = `${this.gap}px`;
      });

      this.$refs.slides.forEach((_, i) => {
        if (i > 0) {
          this.$refs.slides[i].style.transformOrigin = `50% 50% ${-this.apothem}px`;
          this.$refs.slides[i].style.transform = `rotateY(${i * this.theta}rad)`;
        }
      });

      this.rotateCarousel(this.currImage);
    },

    rotateCarousel(imageIndex) {
      if (!this.$refs.figure) {
        return;
      }

      const newRotateY = imageIndex * -this.theta;
      this.$refs.figure.style.transform = `rotateX(-11deg) rotateY(${newRotateY}rad)`;
    },

    onDragStart(event) {
      event.preventDefault();

      this.isDragging = true;
      startPositionX = this.isTouch ? event.touches[0].clientX : event.clientX;

      this.$refs.slides.forEach(slide => {
        slide.addEventListener(this.isTouch ? 'touchmove' : 'mousemove', this.onDrag);
        slide.addEventListener(this.isTouch ? 'touchend' : 'mouseup', this.onDragEnd);
        slide.addEventListener('mouseleave', this.onDragEnd);
      })
    },

    onDragEnd(event) {
      if (!this.isTouch) {
        event.preventDefault();
      }

      this.isDragging = this.isSliding =false;

      this.$refs.slides.forEach(slide => {
        slide.removeEventListener(this.isTouch ? 'touchmove' : 'mousemove', this.onDrag);
        slide.removeEventListener(this.isTouch ? 'touchend' : 'mouseup', this.onDragEnd);
        slide.removeEventListener('mouseleave', this.onDragEnd);
      })

    },

    onDrag(event) {
      if (this.isSliding) {
        return;
      }

      if (!this.isTouch) {
        event.preventDefault();
      }

      const currentPositionX = this.isTouch ? event.touches[0].clientX : event.clientX;
      const deltaX = currentPositionX - startPositionX;

      if (Math.abs(deltaX) > 50) {
        this.isSliding = true;
        let direction = deltaX > 0 ? 'prev' : 'next';
        this.changeSlide(direction);
      }
    },
  },
}
</script>

<style lang="scss" scoped>
// Carousel configuration parameters
$item-width: 40%; // Now we can use percentages!
$item-separation: 0px; // This now is set with Js
$viewer-distance: 500px;

.carousel {
  padding: 20px;

  perspective: $viewer-distance;

  display: flex;
  flex-direction: column;
  align-items: center;

  > * {
    flex: 0 0 auto;
  }

  .figure {
    margin: 0;
    width: $item-width;
    transform-style: preserve-3d;
    transition: transform 0.5s;
    will-change: transform;
    transform: rotateX(-11deg);

    img {
      transition: transform 0.5s;
      will-change: transform;
      width: 100%;
      box-sizing: border-box;
      padding: 0 $item-separation / 2;

      &:not(:first-of-type) {
        position: absolute;
        left: 0;
        top: 0;
      }
    }
  }

  nav {
    display: flex;
    justify-content: center;
    margin: 20px 0 0;

    button {
      flex: 0 0 auto;
      margin: 0 5px;

      cursor: pointer;

      color: #333;
      background: none;
      border: 1px solid;
      letter-spacing: 1px;
      padding: 5px 10px;
    }
  }

  &__slide {
    user-select: none;

    &_grabbing {
      cursor: grabbing;
    }
  }
}
</style>