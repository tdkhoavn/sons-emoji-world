<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const selectedEmojis = ref([])
const zoomEmoji = ref('')
const showFireworks = ref(false)

// Kích thước lưới ma trận 3x2
const gridCols = ref(3)
const gridRows = ref(2)
const capacity = computed(() => gridCols.value * gridRows.value)
const displayedEmojis = computed(() => {
  return selectedEmojis.value.slice(-capacity.value)
})

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

const categoryNames = ['Tất cả', ...Object.keys(categories)]
const selectedCategory = ref('Tất cả')

const allEmojis = Object.values(categories).flat()
const filteredEmojis = computed(() => {
  return selectedCategory.value === 'Tất cả'
    ? allEmojis
    : categories[selectedCategory.value] || []
})

let audioContext = null

function playHappySound() {
  try {
    if (!audioContext) {
      const Ctx = window.AudioContext || window.webkitAudioContext
      audioContext = new Ctx()
    }
    const context = audioContext
    const oscillator = context.createOscillator()
    const gain = context.createGain()

    oscillator.type = 'triangle'
    oscillator.frequency.setValueAtTime(660, context.currentTime)
    oscillator.frequency.exponentialRampToValueAtTime(990, context.currentTime + 0.12)

    gain.gain.setValueAtTime(0.0001, context.currentTime)
    gain.gain.exponentialRampToValueAtTime(0.2, context.currentTime + 0.02)
    gain.gain.exponentialRampToValueAtTime(0.0001, context.currentTime + 0.18)

    oscillator.connect(gain)
    gain.connect(context.destination)
    oscillator.start()
    oscillator.stop(context.currentTime + 0.2)
  } catch (e) {
    // noop if audio not allowed
  }
}

function handleSelect(emoji) {
  // Thêm emoji vào danh sách, giới hạn hiển thị sẽ do displayedEmojis quyết định
  selectedEmojis.value = [...selectedEmojis.value, emoji]
  
  // Hiệu ứng zoom và pháo hoa
  zoomEmoji.value = emoji
  showFireworks.value = true
  
  // Reset sau 500ms
  setTimeout(() => {
    zoomEmoji.value = ''
    showFireworks.value = false
  }, 500)
  
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
              <!-- Overlay mờ khi zoom -->
              <div v-if="zoomEmoji" class="absolute inset-0 bg-black/80 z-5 animate-fade-in"></div>
              
              <!-- Hiệu ứng pháo hoa -->
              <div v-if="showFireworks" class="absolute inset-0 pointer-events-none z-10">
                <div
                  v-for="i in 12"
                  :key="i"
                  class="absolute text-4xl animate-firework"
                  :style="{
                    left: Math.random() * 100 + '%',
                    top: Math.random() * 100 + '%',
                    animationDelay: Math.random() * 0.5 + 's',
                    transform: `rotate(${Math.random() * 360}deg)`
                  }"
                >
                  {{ ['✨', '🎉', '🎊', '⭐', '💫', '🌟'][Math.floor(Math.random() * 6)] }}
                </div>
              </div>
              
              <!-- Emoji zoom to -->
              <div v-if="zoomEmoji" class="absolute inset-0 flex items-center justify-center z-20">
                <span class="text-9xl animate-zoom-bounce drop-shadow-[0_8px_0_rgba(0,0,0,0.3)]">
                  {{ zoomEmoji }}
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
          <!-- Title nhãn hiệu với đám mây trong khung bo -->
          <div class="text-center mb-2 relative">
            <!-- Khung bo bốn góc với background nổi bật -->
            <div class="relative bg-gradient-to-br from-pink-100 via-purple-50 to-blue-100 rounded-3xl p-4 border-2 border-pink-200 shadow-[0_8px_32px_rgba(236,72,153,0.15)] backdrop-blur-sm">
              <!-- Đám mây bồng bềnh -->
              <div class="absolute inset-0 flex items-center justify-center pointer-events-none rounded-3xl overflow-hidden">
                <div class="text-6xl md:text-8xl opacity-30 animate-float">
                  ☁️
                </div>
                <div class="absolute text-4xl md:text-6xl opacity-25 animate-float-delayed" style="left: 20%; top: -10px;">
                  ☁️
                </div>
                <div class="absolute text-3xl md:text-5xl opacity-20 animate-float-slow" style="right: 15%; top: 5px;">
                  ☁️
                </div>
                <div class="absolute text-2xl md:text-4xl opacity-15 animate-float" style="left: 10%; bottom: 10px;">
                  ☁️
                </div>
                <div class="absolute text-3xl md:text-5xl opacity-20 animate-float-slow" style="right: 25%; bottom: 5px;">
                  ☁️
                </div>
              </div>
              
              <!-- Title với hiệu ứng nổi bật -->
              <h1 class="text-2xl md:text-3xl font-bold text-pink-600 drop-shadow-[0_4px_8px_rgba(236,72,153,0.3)] relative z-10" style="font-family: 'Comic Sans MS', cursive, sans-serif;">
                Son's Emoji World
              </h1>
              
              <!-- Hiệu ứng ánh sáng xung quanh title -->
              <div class="absolute inset-0 rounded-3xl bg-gradient-to-r from-pink-200/20 via-purple-200/20 to-blue-200/20 animate-pulse-slow"></div>
            </div>
          </div>
          
          <!-- Segmented control chọn chủ đề -->
          <div class="bg-neutral-100 rounded-full p-1 border border-neutral-300 shadow-[inset_0_2px_0_rgba(255,255,255,0.8)] overflow-x-auto">
            <div class="flex gap-1">
              <button
                v-for="name in categoryNames"
                :key="name"
                class="px-3 py-1.5 rounded-full text-[13px] whitespace-nowrap transition-colors"
                :class="selectedCategory === name
                  ? 'bg-white text-indigo-600 shadow-[0_2px_0_#d4d4d4] border border-neutral-300'
                  : 'text-neutral-700'
                "
                @click="selectedCategory = name"
              >
                {{ name }}
              </button>
            </div>
          </div>

          <!-- Lưới nút emoji dạng phím 3D trong khung bo -->
          <div class="bg-neutral-100 rounded-2xl p-3 border border-neutral-300 shadow-[inset_0_2px_0_rgba(255,255,255,0.8)] overflow-y-auto h-full flex-1 min-h-0">
            <div class="grid grid-cols-5 gap-3">
              <button
                v-for="(emoji, idx) in filteredEmojis"
                :key="idx"
                class="group relative aspect-square rounded-2xl border border-neutral-300 bg-white text-4xl md:text-5xl flex items-center justify-center shadow-[0_6px_0_#c7c7c7,0_14px_26px_rgba(0,0,0,0.18)] active:translate-y-[4px] active:shadow-[0_2px_0_#c7c7c7,0_8px_16px_rgba(0,0,0,0.22)] transition-transform"
                @click="handleSelect(emoji)"
              >
                <span class="translate-y-[-1px]">
                  {{ emoji }}
                </span>
                <!-- ánh sáng trên nút -->
                <span class="pointer-events-none absolute inset-0 rounded-2xl shadow-[inset_0_2px_0_rgba(255,255,255,0.9)]"></span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
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

@keyframes float {
  0%, 100% {
    transform: translateY(0px) rotate(0deg);
  }
  50% {
    transform: translateY(-10px) rotate(5deg);
  }
}

@keyframes float-delayed {
  0%, 100% {
    transform: translateY(0px) rotate(0deg);
  }
  50% {
    transform: translateY(-8px) rotate(-3deg);
  }
}

@keyframes float-slow {
  0%, 100% {
    transform: translateY(0px) rotate(0deg);
  }
  50% {
    transform: translateY(-6px) rotate(2deg);
  }
}

@keyframes pulse-slow {
  0%, 100% {
    opacity: 0.3;
  }
  50% {
    opacity: 0.6;
  }
}

.animate-zoom-bounce {
  animation: zoom-bounce 0.5s ease-out;
}

.animate-firework {
  animation: firework 0.8s ease-out forwards;
}

.animate-fade-in {
  animation: fade-in 0.3s ease-out;
}

.animate-float {
  animation: float 3s ease-in-out infinite;
}

.animate-float-delayed {
  animation: float-delayed 4s ease-in-out infinite;
}

.animate-float-slow {
  animation: float-slow 5s ease-in-out infinite;
}

.animate-pulse-slow {
  animation: pulse-slow 4s ease-in-out infinite;
}
</style>
