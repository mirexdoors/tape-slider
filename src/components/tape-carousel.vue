<template>
  <div class="carousel">
    <nav>
      <button
          class="nav prev"
          @click="changeSlide('next')">Prev</button>
      <button
          class="nav next"
          @click="changeSlide('prev')">Next</button>
    </nav>
    <div
        ref="figure"
        class="figure"
    >
      <img
          v-for="item in items"
          :key="item.Id"
          ref="slides"
          :src="item.Image.LargeUrl">
    </div>
  </div>
</template>

<script>
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
      this.setupCarousel(this.n, parseFloat(getComputedStyle(this.$refs.slides[0]).width));
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

    setupCarousel(n, s) {
      const apothem = s / (2 * Math.tan(Math.PI / this.n));

      this.$refs.figure.style.transformOrigin = `50% 50% ${-apothem}px`;

      this.$refs.slides.forEach((_, i) => {
        this.$refs.slides[i].style.padding = `${this.gap}px`;
      });

      this.$refs.slides.forEach((_, i) => {
        if (i > 0) {
          this.$refs.slides[i].style.transformOrigin = `50% 50% ${-apothem}px`;
          this.$refs.slides[i].style.transform = `rotateY(${i * this.theta}rad)`;
        }
      });

      this.rotateCarousel(this.currImage);
    },

    rotateCarousel(imageIndex) {
      // this.$refs.figure.style.transform = `rotateY(${imageIndex * -this.theta}rad) rotateX(-11deg)`;
      const slides = this.$refs.slides
      const firstSlide = slides.shift()
      slides.push(firstSlide)
      slides.forEach((slide, i)=> {
        slide.style.transform = `rotateY(${i * -this.theta}rad)`
        slide.style.transformOrigin = `50% 50% ${-this.apothem}px`
      })
    }
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
    transform: rotateX(-11deg);

    img {
      transition: transform 0.5s;

      width: 100%;
      box-sizing: border-box;
      padding: 0 $item-separation / 2;

      &:not(:first-of-type) {
        transition: transform 0.5s;

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
}
</style>