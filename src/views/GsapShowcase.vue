<script setup lang="ts">
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { TextPlugin } from 'gsap/TextPlugin'
import { onMounted, onUnmounted, ref } from 'vue'

gsap.registerPlugin(ScrollTrigger, TextPlugin)

// Hero section refs
const heroTitle = ref<HTMLElement>()
const heroSubtitle = ref<HTMLElement>()

// Stagger boxes
const staggerBoxes = ref<HTMLElement[]>([])

// Timeline items
const timelineItems = ref<HTMLElement[]>([])

// ScrollTrigger text
const scrollText = ref<HTMLElement>()

// Parallax refs
const shape1 = ref<HTMLElement>()
const shape2 = ref<HTMLElement>()
const parallaxText = ref<HTMLElement>()

// Text animation refs
const textLines = ref<HTMLElement[]>([])

onMounted(() => {
  // Custom cursor
  const cursor = document.querySelector('.cursor') as HTMLElement
  const cursorDot = document.querySelector('.cursor-dot') as HTMLElement

  if (cursor && cursorDot) {
    gsap.set(cursor, { xPercent: -50, yPercent: -50 })
    gsap.set(cursorDot, { xPercent: -50, yPercent: -50 })

    const handleMouseMove = (e: MouseEvent) => {
      gsap.to(cursor, { duration: 0.3, x: e.clientX, y: e.clientY })
      gsap.to(cursorDot, { duration: 0.1, x: e.clientX, y: e.clientY })
    }

    window.addEventListener('mousemove', handleMouseMove)

    // Store handler for cleanup
    ; (cursor as any)._mousemoveHandler = handleMouseMove
  }

  // Hero animation
  if (heroTitle.value) {
    const letters = heroTitle.value.textContent?.split('') || []
    heroTitle.value.innerHTML = letters
      .map(letter => `<span class="letter">${letter}</span>`)
      .join('')

    gsap.to('.hero-title .letter', {
      opacity: 1,
      y: 0,
      rotateX: 0,
      duration: 1,
      stagger: 0.05,
      ease: 'back.out(1.7)',
    })
  }

  if (heroSubtitle.value) {
    gsap.from(heroSubtitle.value, {
      opacity: 0,
      y: 30,
      duration: 0.8,
      ease: 'power2.out',
      delay: 0.5,
    })
  }

  // Stagger animation
  gsap.from(staggerBoxes.value, {
    scrollTrigger: {
      trigger: '.stagger-section',
      start: 'top 80%',
    },
    y: 100,
    opacity: 0,
    rotation: -45,
    duration: 0.8,
    stagger: 0.1,
    ease: 'back.out(1.7)',
  })

  // Timeline animation - 前4个用普通触发动画
  gsap.to(timelineItems.value.slice(0, 4), {
    x: 0,
    opacity: 1,
    duration: 0.6,
    stagger: 0.2,
    ease: 'power3.out',
    scrollTrigger: {
      trigger: '.timeline-section',
      start: 'top 50%',
    },
  })

  // 后4个用滚动驱动动画
  const timelineScrollSection = gsap.timeline({
    scrollTrigger: {
      trigger: '.timeline-section',
      start: 'top center', // 当section到达屏幕中央时
      end: 'bottom center', // 滚动到section底部到达屏幕中央
      scrub: 1,
    },
  })

  timelineScrollSection.to(timelineItems.value.slice(4, 8), {
    x: 0,
    opacity: 1,
    duration: 1,
    stagger: 0.2,
    ease: 'power2.inOut',
  })

  // ScrollTrigger animation
  if (scrollText.value) {
    gsap.to(scrollText.value, {
      scrollTrigger: {
        trigger: '.scroll-section',
        start: 'top top',
        end: 'bottom top',
        scrub: 1,
      },
      scale: 3,
      rotation: 90,
      color: '#D4774C',
    })
  }

  // Scroll progress bar
  gsap.to('.scroll-progress', {
    scrollTrigger: {
      trigger: 'body',
      start: 'top top',
      end: 'bottom bottom',
      scrub: 0,
    },
    scaleX: 1,
  })

  // Parallax effect - 增强版
  // 背景层：移动最快，创造深度
  if (shape1.value) {
    gsap.to(shape1.value, {
      scrollTrigger: {
        trigger: '.parallax-section',
        start: 'top 50%',
        end: 'bottom top',
        scrub: 0.5, // 减小延迟，响应更直接
      },
      y: -400, // 增加移动距离
      x: 200,
      scale: 1.5, // 添加缩放增强深度感
    })
  }

  // 中景层：反向移动，增强视差
  if (shape2.value) {
    gsap.to(shape2.value, {
      scrollTrigger: {
        trigger: '.parallax-section',
        start: 'top 50%',
        end: 'bottom top',
        scrub: 0.5,
      },
      y: 400, // 反向移动更远
      x: -200,
      scale: 1.3,
    })
  }

  // 前景文字：中等速度，从下方浮入
  if (parallaxText.value) {
    gsap.fromTo(parallaxText.value, {
      y: 150,
      opacity: 0,
      scale: 0.8,
    }, {
      scrollTrigger: {
        trigger: '.parallax-section',
        start: 'top 80%',
        end: 'center center',
        scrub: 0.8,
      },
      y: 0,
      opacity: 1,
      scale: 1,
    })
  }

  // Text reveal animation
  textLines.value.forEach((line) => {
    const lineInner = line.querySelector('.line-inner') as HTMLElement
    if (lineInner) {
      gsap.to(lineInner, {
        scrollTrigger: {
          trigger: line,
          start: 'top 80%',
          end: 'top 20%',
          scrub: 1,
        },
        y: 0,
        duration: 1,
        ease: 'power2.out',
      })
    }
  })
})

onUnmounted(() => {
  // Cleanup
  const cursor = document.querySelector('.cursor') as HTMLElement
  if (cursor && (cursor as any)._mousemoveHandler) {
    window.removeEventListener('mousemove', (cursor as any)._mousemoveHandler)
  }

  // Kill all ScrollTrigger instances
  ScrollTrigger.getAll().forEach(trigger => trigger.kill())
})
</script>

<template>
  <div class="gsap-showcase">
    <!-- Custom Cursor -->
    <div class="cursor" />
    <div class="cursor-dot" />

    <!-- Scroll Progress Bar -->
    <div class="scroll-progress" />

    <!-- Hero Section -->
    <section class="hero">
      <h1 ref="heroTitle" class="hero-title">
        GSAP
      </h1>
      <p ref="heroSubtitle" class="hero-subtitle">
        Animation Showcase
      </p>
      <div class="scroll-indicator">
        <span>Scroll</span>
        <div class="scroll-line" />
      </div>
    </section>

    <!-- Stagger Section -->
    <section class="stagger-section">
      <span class="section-label">01 — Stagger</span>
      <span class="section-number">01</span>

      <div class="stagger-grid">
        <div
          v-for="i in 10" :key="i" :ref="(el) => { if (el) staggerBoxes[i - 1] = el as HTMLElement }"
          class="stagger-box"
        >
          {{ i }}
        </div>
      </div>
    </section>

    <!-- Timeline Section -->
    <section class="timeline-section">
      <span class="section-label">02 — Timeline</span>
      <span class="section-number">02</span>

      <div class="timeline-container">
        <div
          v-for="(_, index) in 8" :key="index" :ref="(el) => { if (el) timelineItems[index] = el as HTMLElement }"
          class="timeline-item"
        >
          <h3>Step {{ ['One', 'Two', 'Three', 'Four', 'Five', 'Six', 'Seven', 'Eight'][index] }}</h3>
          <p>
            {{
              [
                'Sequences are controlled with precision timing. Each animation plays in order.',
                'Timelines let you orchestrate complex animations with simple callbacks.',
                'Nest timelines within timelines for hierarchical control.',
                'Control playback, reverse, and even timeScale for perfect synchronization.',
                'Scroll-driven animations respond to user scrolling behavior in real-time.',
                'The scrub option links animation progress directly to scroll position.',
                'Scroll up to reverse, scroll down to play forward smoothly.',
                'Create immersive experiences that feel natural and responsive.',
              ][index]
            }}
          </p>
        </div>
      </div>
    </section>

    <!-- ScrollTrigger Section -->
    <section class="scroll-section">
      <span class="section-label">03 — ScrollTrigger</span>
      <span class="section-number">03</span>

      <div class="scroll-container">
        <div class="scroll-content">
          <div ref="scrollText" class="scroll-text">
            SCROLL
          </div>
        </div>
      </div>
    </section>

    <!-- Parallax Section -->
    <section class="parallax-section">
      <span class="section-label">04 — Parallax</span>
      <span class="section-number">04</span>

      <div class="parallax-container">
        <div class="parallax-layer">
          <div ref="shape1" class="parallax-shape shape-1" />
          <div ref="shape2" class="parallax-shape shape-2" />
        </div>
        <div class="parallax-content">
          <div ref="parallaxText" class="parallax-text">
            Parallax
          </div>
        </div>
      </div>
    </section>

    <!-- Text Animation Section -->
    <section class="text-section">
      <span class="section-label">05 — Text Animation</span>
      <span class="section-number">05</span>

      <div class="text-container">
        <div class="text-reveal">
          <span
            v-for="(text, index) in ['Animate', 'Everything', 'With GSAP']"
            :key="index"
            :ref="(el) => { if (el) textLines[index] = el as HTMLElement }" class="line"
          >
            <span class="line-inner">{{ text }}</span>
          </span>
        </div>
      </div>
    </section>

    <footer>
      <p>GSAP — <a href="https://gsap.com" target="_blank">Learn More</a></p>
    </footer>
  </div>
</template>

<style scoped>
.gsap-showcase {
  font-family: 'Space Mono', monospace;
  background: #1A1A18;
  color: #F5F3EF;
  cursor: none;
}

/* Custom Cursor */
.cursor {
  width: 20px;
  height: 20px;
  border: 2px solid #D4774C;
  border-radius: 50%;
  position: fixed;
  pointer-events: none;
  z-index: 9999;
  mix-blend-mode: difference;
  transition: transform 0.1s ease;
}

.cursor-dot {
  width: 6px;
  height: 6px;
  background: #4A7C6F;
  border-radius: 50%;
  position: fixed;
  pointer-events: none;
  z-index: 9999;
}

/* Scroll Progress */
.scroll-progress {
  position: fixed;
  top: 0;
  left: 0;
  height: 4px;
  background: #D4774C;
  transform-origin: left;
  z-index: 1000;
  transform: scaleX(0);
  width: 100%;
}

/* Hero Section */
.hero {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  position: relative;
  background:
    linear-gradient(90deg, rgba(212, 119, 76, 0.08) 1px, transparent 1px),
    linear-gradient(rgba(212, 119, 76, 0.08) 1px, transparent 1px);
  background-size: 50px 50px;
  background-position: center center;
}

.hero::before {
  content: '';
  position: absolute;
  inset: 0;
  background: radial-gradient(circle at center, transparent 0%, #1A1A18 70%);
}

.hero-title {
  font-family: 'Bebas Neue', sans-serif;
  font-size: clamp(80px, 20vw, 250px);
  line-height: 0.85;
  text-align: center;
  position: relative;
  z-index: 1;
  color: #F5F3EF;
  text-transform: uppercase;
  letter-spacing: -0.02em;
}

.hero-title .letter {
  display: inline-block;
  opacity: 0;
  transform: translateY(100px) rotateX(-90deg);
}

.hero-subtitle {
  font-size: clamp(14px, 2vw, 20px);
  margin-top: 40px;
  color: #4A7C6F;
  text-transform: uppercase;
  letter-spacing: 0.3em;
  position: relative;
  z-index: 1;
}

.scroll-indicator {
  position: absolute;
  bottom: 60px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  z-index: 1;
}

.scroll-indicator span {
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 0.2em;
  color: #8B8680;
}

.scroll-line {
  width: 1px;
  height: 60px;
  background: #D4774C;
  position: relative;
  overflow: hidden;
}

.scroll-line::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 50%;
  background: #4A7C6F;
  animation: scrollDown 1.5s ease-in-out infinite;
}

@keyframes scrollDown {
  0% {
    transform: translateY(-100%);
  }

  100% {
    transform: translateY(200%);
  }
}

/* Section Styles */
section {
  min-height: 100vh;
  padding: 120px 60px;
  position: relative;
}

.section-label {
  position: absolute;
  top: 40px;
  left: 60px;
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 0.2em;
  color: #8B8680;
}

.section-number {
  position: absolute;
  top: 40px;
  right: 60px;
  font-family: 'Bebas Neue', sans-serif;
  font-size: 120px;
  color: #F5F3EF;
  opacity: 0.05;
  line-height: 1;
}

/* Stagger Section */
.stagger-section {
  background: #1A1A18;
  color: #F5F3EF;
}

.stagger-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 16px;
  margin-top: 80px;
}

.stagger-box {
  aspect-ratio: 1;
  background: linear-gradient(135deg, #D4774C, #4A7C6F);
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Bebas Neue', sans-serif;
  font-size: 48px;
  color: #1A1A18;
  position: relative;
  overflow: hidden;
}

/* Timeline Section */
.timeline-section {
  background: #D4774C;
  color: #1A1A18;
  position: relative;
  overflow: hidden;
}

.timeline-section::before {
  content: 'TIMELINE';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-family: 'Bebas Neue', sans-serif;
  font-size: 400px;
  color: #1A1A18;
  opacity: 0.03;
  white-space: nowrap;
}

.timeline-container {
  position: relative;
  z-index: 1;
  max-width: 800px;
  margin: 0 auto;
}

.timeline-item {
  background: #1A1A18;
  color: #F5F3EF;
  padding: 40px;
  margin-bottom: 20px;
  position: relative;
  opacity: 0;
  transform: translateX(-100px);
}

.timeline-item::before {
  content: '';
  position: absolute;
  left: -20px;
  top: 50%;
  transform: translateY(-50%);
  width: 20px;
  height: 2px;
  background: #C9A84E;
}

.timeline-item h3 {
  font-family: 'Bebas Neue', sans-serif;
  font-size: 28px;
  margin-bottom: 12px;
  color: #C9A84E;
}

.timeline-item p {
  font-size: 14px;
  color: #8B8680;
  line-height: 1.6;
}

/* ScrollTrigger Section */
.scroll-section {
  background: #F5F3EF;
  color: #1A1A18;
  min-height: 300vh;
}

.scroll-container {
  position: sticky;
  top: 0;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.scroll-content {
  text-align: center;
}

.scroll-text {
  font-family: 'Bebas Neue', sans-serif;
  font-size: clamp(60px, 15vw, 200px);
  line-height: 1;
  color: #1A1A18;
  position: relative;
}

/* Parallax Section */
.parallax-section {
  background: #1A1A18;
  color: #F5F3EF;
  overflow: hidden;
}

.parallax-container {
  position: relative;
  height: 100vh;
}

.parallax-layer {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.parallax-shape {
  position: absolute;
  border-radius: 50%;
}

.shape-1 {
  width: 400px;
  height: 400px;
  background: linear-gradient(135deg, #D4774C, #C9A84E);
  top: 10%;
  left: 10%;
  filter: blur(80px);
  opacity: 0.6;
}

.shape-2 {
  width: 300px;
  height: 300px;
  background: linear-gradient(135deg, #4A7C6F, #D4774C);
  bottom: 20%;
  right: 15%;
  filter: blur(60px);
  opacity: 0.5;
}

.parallax-content {
  font-family: 'Bebas Neue', sans-serif;
  font-size: clamp(80px, 12vw, 180px);
  text-align: center;
  color: #F5F3EF;
  text-transform: uppercase;
  line-height: 1;
}

/* Text Section */
.text-section {
  background: #1A1A18;
  color: #F5F3EF;
  display: flex;
  align-items: center;
  justify-content: center;
}

.text-container {
  max-width: 1000px;
}

.text-reveal {
  font-family: 'Bebas Neue', sans-serif;
  font-size: clamp(60px, 10vw, 150px);
  line-height: 1.1;
  text-transform: uppercase;
}

.text-reveal .line {
  display: block;
  overflow: hidden;
}

.text-reveal .line-inner {
  display: block;
  transform: translateY(100%);
}

/* Footer */
footer {
  background: #1A1A18;
  color: #F5F3EF;
  padding: 60px;
  text-align: center;
  border-top: 1px solid rgba(245, 243, 239, 0.1);
}

footer p {
  font-size: 14px;
  color: #8B8680;
}

footer a {
  color: #D4774C;
  text-decoration: none;
}

/* Responsive */
@media (max-width: 768px) {
  section {
    padding: 80px 30px;
  }

  .section-label {
    left: 30px;
  }

  .section-number {
    right: 30px;
    font-size: 80px;
  }

  .stagger-grid {
    grid-template-columns: repeat(3, 1fr);
  }

  .cursor,
  .cursor-dot {
    display: none;
  }

  .gsap-showcase {
    cursor: auto;
  }
}
</style>
