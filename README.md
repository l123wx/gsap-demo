# GSAP 动画笔记

## 核心方法

### 1. gsap.to() - 最常用

从**当前状态**动画到**目标状态**

```javascript
gsap.to('.box', {
  x: 100, // 从当前位置移动到 x=100
  opacity: 1, // 从当前透明度渐变到 1
  duration: 1
})
```

### 2. gsap.from() - 入场动画

从**指定状态**动画到**当前状态**（CSS定义的状态）

```javascript
gsap.from('.box', {
  x: -100, // 从左侧100px开始
  opacity: 0, // 从透明开始
  duration: 1 // 最终停在CSS定义的位置
})
```

### 3. gsap.fromTo() - 完全控制

同时定义**起点和终点**，不依赖CSS状态

```javascript
gsap.fromTo(
  '.box',
  { x: 0, scale: 0, rotation: 0 }, // 明确的起点
  { x: 50, scale: 1.2, rotation: 180 } // 明确的终点
)
```

## Stagger 交错动画

**核心**: 让多个元素**依次**执行相同的动画

```javascript
gsap.to('.box', {
  x: 100,
  duration: 0.8,
  stagger: 0.1 // 关键参数
})
```

**时间线**:
```
元素0: [====动画====] 0.0s 开始
元素1:     [====动画====] 0.1s 开始
元素2:         [====动画====] 0.2s 开始
元素3:             [====动画====] 0.3s 开始
```

**高级用法**:
```javascript
gsap.to('.box', {
  x: 100,
  duration: 0.8,
  stagger: {
    amount: 1, // 总延迟时间1秒，平均分配
    from: 'center', // 从中间开始向两边扩散
    grid: 'auto', // 按网格位置排序
  }
})
```

## Timeline 时间轴的核心

**关键**: 将多个动画按**顺序**组织，方便控制整体

```javascript
const tl = gsap.timeline()

tl.to('.box1', { x: 100, duration: 1 })
  .to('.box2', { y: 100, duration: 1 }) // 默认等上一个结束
  .to('.box3', { rotation: 360 }, '-=0.5') // 提前0.5秒开始
```

**位置控制**:
```javascript
'<' // 在上一个动画开始时开始（同步）
'-=0.5' // 在上一个动画结束前0.5秒开始（重叠）
'+=0.5' // 在上一个动画结束后0.5秒开始（延迟）
'>-0.5' // 在时间轴最后向前0.5秒
```

**为什么用 Timeline**:
1. **序列控制**: 精确控制动画的先后顺序
2. **整体控制**: 可以一起播放、暂停、反转整个序列
3. **时间管理**: 方便调整动画之间的时间关系
4. **标签定位**: 可以跳转到特定时间点

---

## ScrollTrigger 核心理念

### 基础触发 - 一次性动画

```javascript
gsap.to('.box', {
  opacity: 1,
  scrollTrigger: {
    trigger: '.section', // 监听谁
    start: 'top 80%', // 何时触发
  }
})
```

**关键细节**:
1. 当滚动满足条件时，动画**自动播放一次**
2. 播放后就结束了，不会因为滚动而改变
3. **用途**: 元素进入视口时的入场动画

**start 参数解析**:
```
'top 80%' = 元素的top 到达 视口的80%
```
- 第一个值: 元素的位置 (top, center, bottom, 或像素值)
- 第二个值: 视口的位置 (top, center, bottom, 或百分比)

---

### scrub - 滚动驱动动画

**核心区别**: 动画进度**直接绑定**到滚动位置

```javascript
gsap.to('.box', {
  rotation: 360,
  scrollTrigger: {
    trigger: '.section',
    start: 'top center',
    end: 'bottom center',
    scrub: 1 // 关键！
  }
})
```

**scrub 原理**:
```
滚动进度 0% → 动画进度 0%
滚动进度 50% → 动画进度 50%
滚动进度 100% → 动画进度 100%
```

**关键细节**:
1. `scrub: 1` 中的数字是平滑延迟（秒）
2. `scrub: 0` = 完全跟随，无延迟（可能卡顿）
3. `scrub: 1` = 1秒延迟，更平滑
4. **可以反向**: 向上滚动会反向执行动画
5. **可暂停**: 滚动停止，动画就停在当前位置

---

## 实际案例的关键细节

### 1. 文字分割动画

**原理**: 将文字拆分成单独的 span，然后 stagger

```javascript
// 第一步：拆分文字
const letters = heroTitle.value.textContent.split('')
heroTitle.value.innerHTML = letters.map(letter => `<span>${letter}</span>`).join('')

// 第二步：stagger 动画
gsap.to('.hero-title .letter', {
  opacity: 1,
  y: 0,
  rotateX: 0,
  stagger: 0.05 // 每个字母相隔 0.05 秒
})
```

**关键**: 必须先用 JS 拆分，GSAP 才能分别控制每个字符

---

### 2. 视差滚动

**原理**: 不同元素以不同速度滚动

```javascript
// 背景层：移动慢
gsap.to('.background', {
  y: -200, // 向上移动200px
  scrollTrigger: {
    trigger: '.parallax-section',
    start: 'top bottom', // section顶部到视口底部时开始
    end: 'bottom top', // section底部到视口顶部时结束
    scrub: 1,
  }
})

// 前景层：移动快（或不移动）
// 通过不同的 y 值创造深度感
```

**关键细节**:
1. `start` 和 `end` 定义了滚动的"有效范围"
2. 在这个范围内滚动，元素会从 start 状态变化到 end 状态
3. 不同层次用不同的移动距离，产生3D深度感

---

### 3. 可控制的 Tween

**用途**: 需要手动控制播放的动画（如交互式演示）

```javascript
const tween = gsap.to(element, {
  paused: true, // 先暂停，不自动播放
  rotation: 360,
  scale: 1.5,
  duration: 1
})

// 手动控制
tween.play() // 播放
tween.pause() // 暂停
tween.reverse() // 反向播放
tween.restart() // 重新开始
tween.kill() // 销毁
```

**关键细节**:
1. `paused: true` 让动画不自动执行
2. 可以多次调用 play/reverse
3. 可以动态修改属性后重新播放

---

### 4. 动态更新属性

**场景**: 交互式控制面板

```javascript
const tween = gsap.to(element, {
  rotation: 360,
  scale: 1.5,
  duration: 1
})

// 用户调整滑块
tween.vars.rotation = 720 // 修改目标值
tween.invalidate() // 应用修改（必须！）

// 下次播放时就会用新值
tween.restart()
```

**关键**: 修改 `vars` 后必须调用 `invalidate()`，否则修改不生效

---

## Vue 3 集成的关键点

### 1. 获取 DOM 引用

**关键**: 必须等 DOM 挂载后才能动画

```javascript
const boxRef = ref()

onMounted(() => {
  // ✅ 此时 DOM 已渲染，可以动画
  gsap.to(boxRef.value, { x: 100 })
})
```

---

### 2. ref 数组收集

**核心**: 用回调函数收集多个 ref

```vue
<script setup>
const items = ref<HTMLElement[]>([])

onMounted(() => {
  gsap.to(items.value, { x: 100, stagger: 0.1 })
})
</script>

<template>
  <div
    v-for="i in 5" :key="i"
    :ref="(el) => { if (el) items[i - 1] = el as HTMLElement }"
  />
</template>
```

**关键细节**:
1. `if(el)` 判断避免 null 值
2. 类型断言 `as HTMLElement` 满足 TypeScript
3. 数组索引从 0 开始

---

## 性能优化关键

### 1. 批量属性

```javascript
// ❌ 两次重绘
gsap.to('.box', { x: 100 })
gsap.to('.box', { opacity: 1 })

// ✅ 一次重绘
gsap.to('.box', { x: 100, opacity: 1 })
```

**原理**: GSAP 会将同一帧的动画合并，减少重绘

---

### 2. 使用 will-change

```css
.box {
  will-change: transform, opacity;
}
```

**关键**: 提前告诉浏览器哪些属性会变，浏览器会优化

---

## 参考资源

- 官方文档: https://gsap.com/docs/
- ScrollTrigger: https://gsap.com/docs/v3/Plugins/ScrollTrigger/
- 可视化缓动函数: https://gsap.com/ease-visualizer/
