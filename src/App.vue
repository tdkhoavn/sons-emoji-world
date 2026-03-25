<script setup>
import { ref, computed, onMounted, onBeforeUnmount, nextTick } from 'vue'
import emojiNames from './data/emoji-names.json'

const selectedEmojis = ref([])
const zoomEmoji = ref('')
const showFireworks = ref(false)
const heartParticles = ref([])

// Kích thước lưới ma trận 3x2
const gridCols = ref(3)
const gridRows = ref(2)
const capacity = computed(() => gridCols.value * gridRows.value)
const displayedEmojis = computed(() => {
  return selectedEmojis.value.slice(-capacity.value)
})

function getEmojiName(emoji) {
  return emojiNames[emoji] || ''
}

const categories = {
  'Cảm xúc': ['😀','😃','😄','😁','😆','😅','😂','🤣','😊','😇','🙂','🙃','😉','😌','😍','🥰','😘','😗','😙','😚','😋','😛','😝','😜','🤪','🤨','🧐','🤓','😎','🤩','🥳','😏','😒','😞','😔','😟','😕','🙁','☹️','😣','😖','😫','😩','🥺','😢','😭','😤','😠','😡','🤬','🤯','😳','🥵','🥶','😱','😨','😰','😥','😓','🤗','🤔','🤭','🤫','🤥','😶','😐','😑','😬','🙄','😯','😦','😧','😮','😲','🥱','😴','🤤','😪','😵','🤐','🥴','🤢','🤮','🤧','😷','🤒','🤕','🤑','🤠','😈','👿','👹','👺','🤡','💩','👻','💀','☠️','👽','👾','🤖','🎃','😺','😸','😹','😻','😼','😽','🙀','😿','😾'],
  'Động vật': ['🐶','🐱','🐭','🐹','🐰','🦊','🐻','🐼','🐨','🐯','🦁','🐮','🐷','🐽','🐸','🐵','🙈','🙉','🙊','🐒','🦍','🦧','🐕','🐩','🦮','🐕‍🦺','🐈','🐓','🦃','🦚','🦜','🦢','🦩','🕊️','🐇','🐿️','🦔','🦝','🦨','🦡','🦦','🦥','🐁','🐀','🐿️','🦔','🐾','🐉','🐲','🦕','🦖','🐳','🐋','🐬','🦭','🐟','🐠','🐡','🦈','🐙','🐚','🐌','🦋','🐛','🐜','🐝','🐞','🦗','🕷️','🕸️','🦂','🦟','🦠','🐢','🐍','🦎','🦖','🦕','🐙','🦑','🦐','🦞','🦀','🐡','🐠','🐟','🐬','🐳','🐋','🦭','🐊','🐅','🐆','🦓','🦍','🦧','🐘','🦛','🦏','🐪','🐫','🦒','🦘','🐃','🐂','🐄','🐎','🐖','🐏','🐑','🦙','🐐','🦌','🐕','🐩','🐈','🐓','🦃','🦚','🦜','🦢','🦩','🕊️','🐇','🐿️','🦔','🦝','🦨','🦡','🦦','🦥','🐁','🐀','🐾'],
  'Thức ăn': ['🍎','🍐','🍊','🍋','🍌','🍉','🍇','🍓','🫐','🍈','🍒','🍑','🥭','🍍','🥥','🥝','🍅','🍆','🥑','🥦','🥬','🥒','🌶️','🫒','🌽','🥕','🫑','🥔','🍠','🥐','🥖','🍞','🥨','🥯','🧀','🥚','🍳','🧈','🥞','🧇','🥓','🥩','🍗','🍖','🦴','🌭','🍔','🍟','🍕','🫓','🥙','🌮','🌯','🫔','🥗','🥘','🫕','🥫','🍝','🍜','🍲','🍛','🍣','🍱','🥟','🦪','🍤','🍙','🍚','🍘','🍥','🥠','🥮','🍢','🍡','🍧','🍨','🍦','🥧','🧁','🍰','🎂','🍮','🍭','🍬','🍫','🍿','🍩','🍪','🌰','🥜','🍯'],
  'Phương tiện': ['🚗','🚕','🚙','🚌','🚎','🏎️','🚓','🚑','🚒','🚐','🛻','🚚','🚛','🚜','🏍️','🛵','🚲','🛴','🛹','🛼','🚁','✈️','🛩️','🛫','🛬','🪂','💺','🚀','🛸','🚉','🚞','🚝','🚄','🚅','🚈','🚂','🚆','🚇','🚊','🚍','🚘','🚖','🚡','🚠','🚟','🎢','🎡','🎠','⛵','🛥️','🚤','⛴️','🛳️','🚢','⚓','🚧','⛽','🚨','🚥','🚦','🚏','🗺️','🗿','🗽','🗼','🏰','🏯','🏟️','🎡','🎢','🎠','⛲','⛱️','🏖️','🏝️','🏔️','⛰️','🌋','🗻','🏕️','⛺','🏠','🏡','🏘️','🏚️','🏗️','🏭','🏢','🏬','🏣','🏤','🏥','🏦','🏨','🏪','🏫','🏩','💒','🏛️','⛪','🕌','🕍','🛕','🕋','⛩️','🛤️','🛣️','🗾','🎑','🏞️','🌅','🌄','🌠','🎇','🎆','🌇','🌆','🏙️','🌃','🌌','🌉','🌁'],
  'Thiên nhiên': ['🌱','🌿','☘️','🍀','🎍','🪴','🎋','🍃','🍂','🍁','🍄','🐚','🪨','🌾','💐','🌷','🌹','🥀','🌺','🌸','🌼','🌻','🌞','🌝','🌛','🌜','🌚','🌕','🌖','🌗','🌘','🌑','🌒','🌓','🌔','🌙','⭐','🌟','💫','✨','☄️','☀️','🌤️','⛅','🌥️','☁️','🌦️','🌧️','⛈️','🌩️','🌨️','❄️','☃️','⛄','🌬️','💨','💧','💦','☔','☂️','🌂','🌊','🌫️'],
  'Thể thao': ['⚽','🏀','🏈','⚾','🥎','🎾','🏐','🏉','🎱','🪀','🏓','🏸','🏒','🏑','🥍','🏏','🪃','🥅','⛳','🪁','🏹','🎣','🤿','🥊','🥋','🎽','🛹','🛷','⛸️','🥌','🎿','⛷️','🏂','🪂','🏋️‍♀️','🏋️','🏋️‍♂️','🤼‍♀️','🤼','🤼‍♂️','🤸‍♀️','🤸','🤸‍♂️','⛹️‍♀️','⛹️','⛹️‍♂️','🤺','🤾‍♀️','🤾','🤾‍♂️','🏌️‍♀️','🏌️','🏌️‍♂️','🏇','🧘‍♀️','🧘','🧘‍♂️','🏄‍♀️','🏄','🏄‍♂️','🏊‍♀️','🏊','🏊‍♂️','🤽‍♀️','🤽','🤽‍♂️','🚣‍♀️','🚣','🚣‍♂️','🧗‍♀️','🧗','🧗‍♂️','🚵‍♀️','🚵','🚵‍♂️','🚴‍♀️','🚴','🚴‍♂️','🏆','🥇','🥈','🥉','🏅','🎖️','🏵️','🎗️','🎫','🎟️','🎪','🤹','🤹‍♂️','🤹‍♀️','🎭','🩰','🎨','🎬','🎤','🎧','🎼','🎵','🎶','🪘','🥁','🎹','🎷','🎺','🎸','🪕','🎻','🎲','♠️','♥️','♦️','♣️','🃏','🀄','🎴','🎯','🎳','🎮','🕹️','🎰','🧩','🪅','🎲','♟️','🎯','🎳','🎮','🕹️','🎰','🧩','🪅'],
  'Đồ vật': ['⌚','📱','📲','💻','⌨️','🖥️','🖨️','🖱️','🖲️','💽','💾','💿','📀','🧮','🎥','📷','📸','📹','📼','🔍','🔎','🕯️','💡','🔦','🏮','🪔','📔','📕','📖','📗','📘','📙','📚','📓','📒','📃','📜','📄','📰','🗞️','📑','🔖','🏷️','💰','💴','💵','💶','💷','💸','💳','💎','⚖️','🧰','🔧','🔨','⚒️','🛠️','⛏️','🔩','⚙️','🧱','⛓️','🧲','🔫','💣','🧨','🪓','🔪','🗡️','⚔️','🛡️','🚬','⚰️','🪦','⚱️','🏺','🔮','📿','🧿','💈','⚗️','🔭','🔬','🕳️','🩹','🩺','💊','💉','🩸','🧬','🦠','🧫','🧪','🌡️','🧹','🧺','🧻','🚽','🚰','🚿','🛁','🛀','🧼','🪥','🧽','🪣','🧴','🧷','🧸','🪆','🖼️','🪞','🪟','🛋️','🛏️','🛌','🖼️','🪞','🪟','🛋️','🛏️','🛌','🖼️','🪞','🪟','🛋️','🛏️','🛌'],
  'Biểu tượng': ['❤️','🧡','💛','💚','💙','💜','🖤','🤍','🤎','💔','❣️','💕','💞','💓','💗','💖','💘','💝','💟','☮️','✝️','☪️','🕉️','☸️','✡️','🔯','🕎','☯️','☦️','🛐','⛎','♈','♉','♊','♋','♌','♍','♎','♏','♐','♑','♒','♓','🆔','⚛️','🉑','☢️','☣️','📴','📳','🈶','🈚','🈸','🈺','🈷️','✴️','🆚','💮','🉐','㊙️','㊗️','🈴','🈵','🈹','🈲','🅰️','🅱️','🆎','🆑','🅾️','🆘','❌','⭕','🛑','⛔','📛','🚫','💯','💢','♨️','🚷','🚯','🚳','🚱','🔞','📵','🚭','❗','❕','❓','❔','‼️','⁉️','🔅','🔆','〽️','⚠️','🚸','🔱','⚜️','🔰','♻️','✅','🈯','💹','❇️','✳️','❎','🌐','💠','Ⓜ️','🌀','💤','🏧','🚾','♿','🅿️','🈳','🈂️','🛂','🛃','🛄','🛅','🚹','🚺','🚼','🚻','🚮','🎦','📶','🈁','🔣','ℹ️','🔤','🔡','🔠','🆖','🆗','🆙','🆒','🆕','🆓','0️⃣','1️⃣','2️⃣','3️⃣','4️⃣','5️⃣','6️⃣','7️⃣','8️⃣','9️⃣','🔟']
}

const categoryNames = Object.keys(categories)
const selectedCategory = ref(categoryNames[0])
const categoryScrollRef = ref(null)
const canScrollLeft = ref(false)
const canScrollRight = ref(true)

function updateScrollIndicators() {
  const el = categoryScrollRef.value
  if (!el) return
  canScrollLeft.value = el.scrollLeft > 4
  canScrollRight.value = el.scrollLeft < el.scrollWidth - el.clientWidth - 4
}

const allEmojis = Object.values(categories).flat()
const filteredEmojis = computed(() => {
  return selectedCategory.value === 'Tất cả'
    ? allEmojis
    : categories[selectedCategory.value] || []
})

// Tạo âm thanh "ka-ching" cash register dạng WAV blob - hoạt động trên iOS
let soundUrl = null

function generateCashingWav() {
  const sampleRate = 44100
  const duration = 0.45
  const numSamples = Math.floor(sampleRate * duration)
  const buffer = new ArrayBuffer(44 + numSamples * 2)
  const view = new DataView(buffer)

  // WAV header
  const writeStr = (offset, str) => { for (let i = 0; i < str.length; i++) view.setUint8(offset + i, str.charCodeAt(i)) }
  writeStr(0, 'RIFF')
  view.setUint32(4, 36 + numSamples * 2, true)
  writeStr(8, 'WAVE')
  writeStr(12, 'fmt ')
  view.setUint32(16, 16, true)
  view.setUint16(20, 1, true)
  view.setUint16(22, 1, true)
  view.setUint32(24, sampleRate, true)
  view.setUint32(28, sampleRate * 2, true)
  view.setUint16(32, 2, true)
  view.setUint16(34, 16, true)
  writeStr(36, 'data')
  view.setUint32(40, numSamples * 2, true)

  for (let i = 0; i < numSamples; i++) {
    const t = i / sampleRate
    let sample = 0

    // Phase 1: "ka" - tiếng kim loại chạm nhau (0 - 0.06s)
    if (t < 0.06) {
      const hitEnv = (t < 0.003 ? t / 0.003 : 1) * Math.exp(-t * 40)
      // Noise ngắn + tone cao mô phỏng tiếng gõ kim loại
      const noise = (Math.sin(t * 3500) * 0.5 + Math.sin(t * 5200) * 0.3 + Math.sin(t * 7800) * 0.2)
      sample += noise * hitEnv * 0.6
    }

    // Phase 2: "ching" - tiếng chuông ngân (0.04s - hết)
    if (t > 0.04) {
      const t2 = t - 0.04
      const bellEnv = (t2 < 0.008 ? t2 / 0.008 : 1) * Math.exp(-t2 * 5)
      // Hòa âm chuông: fundamental + overtones
      const bell = Math.sin(2 * Math.PI * 2200 * t2) * 0.4
        + Math.sin(2 * Math.PI * 3300 * t2) * 0.25
        + Math.sin(2 * Math.PI * 4400 * t2) * 0.15
        + Math.sin(2 * Math.PI * 5500 * t2) * 0.1
        + Math.sin(2 * Math.PI * 6600 * t2) * 0.05
        + Math.sin(2 * Math.PI * 8800 * t2) * 0.05
      sample += bell * bellEnv * 0.7
    }

    // Phase 3: tiếng ngân dài nhẹ (shimmer)
    if (t > 0.08) {
      const t3 = t - 0.08
      const shimmerEnv = Math.exp(-t3 * 3.5)
      const shimmer = Math.sin(2 * Math.PI * 1100 * t3) * 0.3
        + Math.sin(2 * Math.PI * 2200 * t3) * 0.2
      sample += shimmer * shimmerEnv * 0.3
    }

    sample = Math.max(-1, Math.min(1, sample))
    view.setInt16(44 + i * 2, sample * 0x7FFF, true)
  }

  const blob = new Blob([buffer], { type: 'audio/wav' })
  return URL.createObjectURL(blob)
}

function playHappySound() {
  try {
    if (!soundUrl) soundUrl = generateCashingWav()
    const audio = new Audio(soundUrl)
    audio.volume = 0.6
    audio.play().catch(() => {})
  } catch (e) {
    // noop
  }
}

function handleSelect(emoji) {
  selectedEmojis.value = [...selectedEmojis.value, emoji]
  
  // Hiệu ứng rơi + trái tim
  zoomEmoji.value = emoji
  showFireworks.value = true
  
  // Tạo các hạt trái tim/ngôi sao bay tỏa ra
  const particles = []
  const icons = ['💖', '💗', '✨', '⭐', '💕', '🌟', '💝', '💫']
  for (let i = 0; i < 10; i++) {
    const angle = (i / 10) * 360
    const distance = 60 + Math.random() * 40
    particles.push({
      id: Date.now() + i,
      icon: icons[Math.floor(Math.random() * icons.length)],
      x: Math.cos(angle * Math.PI / 180) * distance,
      y: Math.sin(angle * Math.PI / 180) * distance,
      delay: Math.random() * 0.15,
      size: 0.6 + Math.random() * 0.6,
    })
  }
  heartParticles.value = particles
  
  setTimeout(() => {
    zoomEmoji.value = ''
    showFireworks.value = false
    heartParticles.value = []
  }, 800)
  
  playHappySound()
}

function clearSelection() {
  if (selectedEmojis.value.length === 0) return
  selectedEmojis.value = selectedEmojis.value.slice(0, -1)
}

function randomSelect() {
  const pool = filteredEmojis.value
  if (pool.length === 0) return
  const randomIndex = Math.floor(Math.random() * pool.length)
  handleSelect(pool[randomIndex])
}

// Scroll-based category detection
const emojiGridRef = ref(null)
const categorySectionRefs = ref({})
const tabRefs = ref({})
const isScrollingToCategory = ref(false)

function setCategoryRef(name, el) {
  if (el) categorySectionRefs.value[name] = el
}

function setTabRef(name, el) {
  if (el) tabRefs.value[name] = el
}

function scrollTabIntoView(name) {
  const tab = tabRefs.value[name]
  if (tab) {
    tab.scrollIntoView({ behavior: 'smooth', block: 'nearest', inline: 'center' })
    setTimeout(updateScrollIndicators, 350)
  }
}

function handleCategoryClick(name) {
  selectedCategory.value = name
  const el = categorySectionRefs.value[name]
  if (el && emojiGridRef.value) {
    isScrollingToCategory.value = true
    const containerRect = emojiGridRef.value.getBoundingClientRect()
    const elRect = el.getBoundingClientRect()
    const offset = elRect.top - containerRect.top + emojiGridRef.value.scrollTop
    emojiGridRef.value.scrollTo({ top: offset, behavior: 'smooth' })
    setTimeout(() => { isScrollingToCategory.value = false }, 600)
  }
  scrollTabIntoView(name)
}

function handleEmojiScroll() {
  if (isScrollingToCategory.value) return
  const container = emojiGridRef.value
  if (!container) return

  const containerTop = container.getBoundingClientRect().top
  const categoryKeys = Object.keys(categories)
  let activeCategory = categoryKeys[0]

  for (const name of categoryKeys) {
    const el = categorySectionRefs.value[name]
    if (!el) continue
    const rect = el.getBoundingClientRect()
    if (rect.top <= containerTop + 40) {
      activeCategory = name
    }
  }

  if (selectedCategory.value !== activeCategory) {
    selectedCategory.value = activeCategory
    scrollTabIntoView(activeCategory)
  }
}

onMounted(() => {
  nextTick(() => {
    emojiGridRef.value?.addEventListener('scroll', handleEmojiScroll, { passive: true })
    updateScrollIndicators()
  })

  // Pre-generate sound
  soundUrl = generateCashingWav()
})

onBeforeUnmount(() => {
  emojiGridRef.value?.removeEventListener('scroll', handleEmojiScroll)
})
</script>

<template>
  <!-- Nền mềm dạng gradient nhẹ -->
  <div class="min-h-dvh w-full flex items-center justify-center bg-[linear-gradient(135deg,#eff6ff,40%,#ffe4e6)] text-gray-900 touch-manipulation select-none p-3">
    <!-- Vỏ máy game 3D -->
    <div class="w-full max-w-md aspect-[9/16] bg-neutral-200 rounded-[28px] border border-neutral-300 p-3 relative shadow-[inset_0_3px_0_rgba(255,255,255,0.7),0_18px_40px_rgba(0,0,0,0.25)]">
      <!-- Dải loa nhỏ trên cùng -->
      <div class="absolute left-1/2 -translate-x-1/2 top-2 flex gap-1 opacity-50">
        <span v-for="i in 6" :key="i" class="w-1.5 h-1.5 rounded-full bg-neutral-400"></span>
      </div>

      <!-- Thân trong chia 30% / 70% -->
      <div class="h-full w-full flex flex-col">
        <!-- Màn hình (30%) với viền bezel đen -->
        <div class="basis-[30%] p-2">
          <div class="w-full h-full bg-black rounded-2xl shadow-[inset_0_10px_30px_rgba(0,0,0,0.6)] p-2">
            <div class="w-full h-full rounded-xl bg-neutral-900/95 flex flex-col items-center justify-center gap-3 shadow-[inset_0_8px_16px_rgba(255,255,255,0.06)] p-2 relative overflow-hidden">
              <!-- Overlay mờ khi hiệu ứng -->
              <div v-if="zoomEmoji" class="absolute inset-0 bg-black/70 z-5 animate-fade-in"></div>
              
              <!-- Hạt trái tim / ngôi sao bay tỏa ra -->
              <div v-if="showFireworks" class="absolute inset-0 pointer-events-none z-30 flex items-center justify-center">
                <div
                  v-for="p in heartParticles"
                  :key="p.id"
                  class="absolute animate-heart-burst"
                  :style="{
                    '--tx': p.x + 'px',
                    '--ty': p.y + 'px',
                    animationDelay: p.delay + 's',
                    fontSize: (p.size * 1.5) + 'rem',
                  }"
                >
                  {{ p.icon }}
                </div>
              </div>
              
              <!-- Emoji rơi + nảy -->
              <div v-if="zoomEmoji" class="absolute inset-0 flex flex-col items-center justify-center z-20">
                <span class="text-9xl animate-bouncy-drop drop-shadow-[0_10px_0_rgba(0,0,0,0.4)]">
                  {{ zoomEmoji }}
                </span>
                <span v-if="getEmojiName(zoomEmoji)" class="mt-2 text-white text-lg font-bold animate-fade-in-up drop-shadow-[0_2px_4px_rgba(0,0,0,0.8)]" style="font-family: 'Comic Neue', 'Comic Sans MS', cursive;">
                  {{ getEmojiName(zoomEmoji) }}
                </span>
              </div>
              
              <div class="w-full h-full grid grid-cols-3 grid-rows-2 gap-3 justify-items-center overflow-hidden">
                <span v-if="displayedEmojis.length === 0" class="text-7xl leading-none drop-shadow-[0_6px_0_rgba(0,0,0,0.15)] self-center">✨</span>
                <span
                  v-for="(e, i) in displayedEmojis"
                  :key="i"
                  class="text-6xl leading-none aspect-square flex items-center justify-center"
                >
                  {{ e }}
                </span>
              </div>
              <div class="flex gap-2 justify-center">
                <button class="px-3 py-1.5 rounded-lg bg-indigo-600 text-white shadow-[0_3px_0_#4f46e5] active:translate-y-[1px] active:shadow-[0_1px_0_#4f46e5] text-sm" @click="randomSelect">Ngẫu nhiên</button>
                <button class="px-3 py-1.5 rounded-lg bg-neutral-100 border border-neutral-300 shadow-[0_3px_0_#c7c7c7] active:translate-y-[1px] active:shadow-[0_1px_0_#c7c7c7] text-sm" @click="clearSelection">Xóa</button>
              </div>
            </div>
          </div>
        </div>

        <!-- Khu phím (70%)) -->
        <div class="basis-[70%] p-2 pt-0 flex flex-col gap-3 min-h-0">
          <!-- Title khắc chìm vào vỏ máy -->
          <h1 class="engraved-text text-center">
            Son's Emoji World
          </h1>
          
          <!-- Segmented control chọn chủ đề -->
          <div class="category-tabs-wrapper relative">
            <div class="fade-left" :class="{ visible: canScrollLeft }"></div>
            <div class="fade-right" :class="{ visible: canScrollRight }"></div>
          <div ref="categoryScrollRef" class="bg-neutral-100 rounded-full p-1 border border-neutral-300 shadow-[inset_0_2px_0_rgba(255,255,255,0.8)] overflow-x-auto hide-scrollbar" @scroll="updateScrollIndicators">
            <div class="flex gap-1">
              <button
                v-for="name in categoryNames"
                :key="name"
                :ref="el => setTabRef(name, el)"
                class="px-3 py-1.5 rounded-full text-[13px] whitespace-nowrap transition-colors"
                :class="selectedCategory === name
                  ? 'bg-white text-indigo-600 shadow-[0_2px_0_#d4d4d4] border border-neutral-300'
                  : 'text-neutral-700'
                "
                @click="handleCategoryClick(name)"
              >
                {{ name }}
              </button>
            </div>
          </div>
          </div>

          <!-- Lưới nút emoji dạng phím 3D trong khung bo -->
          <div class="relative flex-1 min-h-0">
          <div ref="emojiGridRef" class="bg-neutral-100 rounded-2xl p-3 border border-neutral-300 shadow-[inset_0_2px_0_rgba(255,255,255,0.8)] overflow-y-auto h-full">
            <template v-for="(emojis, catName, catIdx) in categories" :key="catName">
              <div
                :ref="el => setCategoryRef(catName, el)"
                class="h-0 w-0 overflow-hidden"
              ></div>
              <div :class="['grid grid-cols-5 gap-3', catIdx > 0 ? 'mt-3' : '']">
                <button
                  v-for="(emoji, idx) in emojis"
                  :key="idx"
                  class="group relative aspect-square rounded-2xl border border-neutral-300 bg-white text-4xl md:text-5xl flex items-center justify-center shadow-[0_6px_0_#c7c7c7,0_14px_26px_rgba(0,0,0,0.18)] active:translate-y-[4px] active:shadow-[0_2px_0_#c7c7c7,0_8px_16px_rgba(0,0,0,0.22)] transition-transform"
                  @click="handleSelect(emoji)"
                >
                  <span class="translate-y-[-1px]">
                    {{ emoji }}
                  </span>
                  <span class="pointer-events-none absolute inset-0 rounded-2xl shadow-[inset_0_2px_0_rgba(255,255,255,0.9)]"></span>
                </button>
              </div>
            </template>
          </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@keyframes bouncy-drop {
  0% {
    transform: translateY(-120%) scale(0.3) rotate(-10deg);
    opacity: 0;
  }
  25% {
    opacity: 1;
  }
  50% {
    transform: translateY(8%) scale(1.15) rotate(3deg);
  }
  65% {
    transform: translateY(-12%) scale(0.95) rotate(-2deg);
  }
  78% {
    transform: translateY(4%) scale(1.05) rotate(1deg);
  }
  88% {
    transform: translateY(-3%) scale(0.98);
  }
  100% {
    transform: translateY(0) scale(1) rotate(0deg);
    opacity: 1;
  }
}

@keyframes heart-burst {
  0% {
    transform: translate(0, 0) scale(0);
    opacity: 0;
  }
  20% {
    opacity: 1;
    transform: translate(0, 0) scale(1);
  }
  100% {
    transform: translate(var(--tx), var(--ty)) scale(0.3);
    opacity: 0;
  }
}

@keyframes zoom-bounce {
  0% {
    transform: scale(0.5);
    opacity: 0;
  }
  50% {
    transform: scale(1.2);
    opacity: 1;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

@keyframes firework {
  0% {
    transform: scale(0) rotate(0deg);
    opacity: 1;
  }
  50% {
    transform: scale(1) rotate(180deg);
    opacity: 0.8;
  }
  100% {
    transform: scale(0.5) rotate(360deg);
    opacity: 0;
  }
}

@keyframes fade-in {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}



.animate-zoom-bounce {
  animation: zoom-bounce 0.5s ease-out;
}

.animate-bouncy-drop {
  animation: bouncy-drop 0.7s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}

.animate-heart-burst {
  animation: heart-burst 0.7s ease-out forwards;
}

.animate-firework {
  animation: firework 0.8s ease-out forwards;
}

.animate-fade-in {
  animation: fade-in 0.3s ease-out;
}

.animate-fade-in-up {
  animation: fade-in-up 0.4s ease-out 0.3s both;
}

@keyframes fade-in-up {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.engraved-text {
  font-family: 'Black Ops One', cursive;
  font-size: 1.15rem;
  font-weight: 400;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: oklch(55% 0 0);
  text-shadow:
    0 1px 1px rgba(255,255,255,0.9),
    0 -1px 1px rgba(0,0,0,0.3);
  margin-bottom: 0.25rem;
}

.fade-up-enter-active,
.fade-up-leave-active {
  transition: opacity 0.2s ease, transform 0.2s ease;
}

.fade-up-enter-from,
.fade-up-leave-to {
  opacity: 0;
  transform: translateY(8px);
}

.hide-scrollbar {
  -ms-overflow-style: none;
  scrollbar-width: none;
}
.hide-scrollbar::-webkit-scrollbar {
  display: none;
}

.category-tabs-wrapper {
  position: relative;
}

.fade-left,
.fade-right {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 24px;
  z-index: 2;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.2s ease;
}
.fade-left.visible,
.fade-right.visible {
  opacity: 1;
}
.fade-left {
  left: 0;
  background: linear-gradient(to right, oklch(0.97 0 0), transparent);
  border-radius: 9999px 0 0 9999px;
}
.fade-right {
  right: 0;
  background: linear-gradient(to left, oklch(0.97 0 0), transparent);
  border-radius: 0 9999px 9999px 0;
}
</style>
