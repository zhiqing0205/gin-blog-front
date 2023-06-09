<script setup lang="ts">
// 文章 ref 对象
const { preview } = defineProps<{ preview: any }>()

onMounted(() => {
  // 当preview加载完成后, 才能获取到锚点目录
  let timer = setInterval(() => {
    if (preview.$el) {
      buildAnchorTitles()
      clearInterval(timer)
    }
  }, 100)
})

let titles = $ref<any>([]) // 锚点目录
let currentIdx = $ref(0) // 当前激活的锚点索引

function buildAnchorTitles() {
  const anchors = preview.$el.querySelectorAll('h1,h2,h3,h4,h5,h6')
  const titleList = Array.from(anchors).filter((t: any) => !!t.innerText.trim())
  if (!titleList.length)
    titles = []
  const hTags = Array.from(new Set(titleList.map((t: any) => t.tagName))).sort()

  let count = [0, 0, 0, 0, 0, 0]
  titles = titleList.map((el: any, idx: number) => {
    // 获取元素的级别
    const level = hTags.indexOf(el.tagName)
    // 当前级别的索引加1
    const index = ++count[level]

    // 重置低级别的索引
    for(let i = level + 1; i < count.length; i++)
      count[i] = 0

    // 拼接索引号
    const number = count.slice(0, level + 1).join('.')
    // 更新当前元素的计数器
    count[level-1] = index;

    return {
      title: el.innerText,
      lineIndex: el.getAttribute('data-line'),
      indent: hTags.indexOf(el.tagName),
      number: number,
    }
  })
}

// 点击锚点目录
function handleAnchorClick(anchor: any, idx: number) {
  const heading = preview.$el.querySelector(`[data-line="${anchor.lineIndex}"]`)
  // const heading = preview.querySelector(`#${anchor.title}`)
  if (heading) {
    window.scrollTo({
      behavior: 'smooth',
      top: heading.offsetTop - 40,
    })
    // preview.scrollToTarget({
    //   target: heading,
    //   scrollContainer: window,
    //   top: 40,
    // })
    setTimeout(() => currentIdx = idx, 600)
  }
}

// * 实现目录高亮当前位置的标题
// 思路: 循环的方式将标题距离顶部距离与滚动条当前位置对比, 来确定高亮的标题
const { y } = $(useWindowScroll())
watchThrottled($$(y), () => {
  titles.forEach((e: any, idx: number) => {
    const heading = preview.$el.querySelector(`[data-line="${e.lineIndex}"]`)
    if (y >= heading.offsetTop - 50) // 比 40 稍微多一点
      currentIdx = idx
  })
}, { throttle: 200 })
</script>

<template>
  <Transition name="slide-fade" appear>
    <div card-view mb-15>
      <div flex items-center mb-10 text-18>
        <i-fa-solid:list-ul />
        <span ml-10>目录</span>
      </div>
      <div
        v-for="(anchor, idx) of titles" :key="anchor.title"
        py-4 rounded-1 border-l-4 border-transparent color="#666261"
        :class="currentIdx === idx ? 'bg-#00c4b6 text-white border-l-#009d92' : '' "
        :style="{ paddingLeft: `${5 + anchor.indent * 15}px` }"
        @click="handleAnchorClick(anchor, idx)"
      >
         <button> {{ anchor.number }} {{ anchor.title }} </button>
      </div>
    </div>
  </Transition>
</template>
