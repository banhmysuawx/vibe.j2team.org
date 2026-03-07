<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { RouterLink } from 'vue-router'
import { Game } from './engine/Game'

const canvasRef = ref<HTMLCanvasElement>()
let game: Game | null = null

const GAME_WIDTH = 800
const GAME_HEIGHT = 480

const isMuted = ref(false)
const isMobile = ref(false)
const activeTab = ref<'play' | 'boss'>('play')
const guideLang = ref<'vi' | 'en'>('vi')

function checkMobile() {
  isMobile.value = 'ontouchstart' in window || window.innerWidth < 768
}

function handleResize() {
  checkMobile()
}

function toggleMute() {
  if (game) {
    isMuted.value = game.toggleMute()
  }
}

// Touch control handlers
function onTouchBtn(action: string, pressed: boolean) {
  if (!game) return
  switch (action) {
    case 'left':
      game.setTouchLeft(pressed)
      break
    case 'right':
      game.setTouchRight(pressed)
      break
    case 'jump':
      game.setTouchJump(pressed)
      break
  }
}

function onCanvasTap() {
  if (game) {
    game.handleTap()
  }
}

onMounted(() => {
  checkMobile()
  window.addEventListener('resize', handleResize)

  if (canvasRef.value) {
    canvasRef.value.width = GAME_WIDTH
    canvasRef.value.height = GAME_HEIGHT
    game = new Game(canvasRef.value)
    game.start()
  }
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  if (game) {
    game.stop()
    game = null
  }
})
</script>

<template>
  <div class="mario-game">
    <!-- Top bar -->
    <div class="top-bar">
      <RouterLink to="/" class="back-link">&larr; Trang chủ</RouterLink>
      <span class="game-title">Super Mario - Boss Battle</span>
      <button class="mute-btn" @click="toggleMute">
        {{ isMuted ? 'SOUND OFF' : 'SOUND ON' }}
      </button>
    </div>

    <!-- Canvas container -->
    <div class="canvas-wrap">
      <canvas
        ref="canvasRef"
        class="game-canvas"
        @click="onCanvasTap"
        @touchstart.prevent="onCanvasTap"
      />
    </div>

    <!-- Mobile touch controls -->
    <div v-if="isMobile" class="touch-controls">
      <div class="dpad">
        <button
          class="touch-btn left-btn"
          @touchstart.prevent="onTouchBtn('left', true)"
          @touchend.prevent="onTouchBtn('left', false)"
          @touchcancel.prevent="onTouchBtn('left', false)"
        >
          &larr;
        </button>
        <button
          class="touch-btn right-btn"
          @touchstart.prevent="onTouchBtn('right', true)"
          @touchend.prevent="onTouchBtn('right', false)"
          @touchcancel.prevent="onTouchBtn('right', false)"
        >
          &rarr;
        </button>
      </div>
      <button
        class="touch-btn jump-btn"
        @touchstart.prevent="onTouchBtn('jump', true)"
        @touchend.prevent="onTouchBtn('jump', false)"
        @touchcancel.prevent="onTouchBtn('jump', false)"
      >
        JUMP
      </button>
    </div>

    <!-- Guide Section -->
    <div class="guide-section">
      <!-- Tab headers -->
      <div class="guide-tabs">
        <button
          class="tab-btn"
          :class="{ active: activeTab === 'play' }"
          @click="activeTab = 'play'"
        >
          {{ guideLang === 'vi' ? 'Cách chơi' : 'How to Play' }}
        </button>
        <button
          class="tab-btn"
          :class="{ active: activeTab === 'boss' }"
          @click="activeTab = 'boss'"
        >
          {{ guideLang === 'vi' ? 'Đánh Boss' : 'Boss Fight' }}
        </button>
        <button class="lang-toggle" @click="guideLang = guideLang === 'vi' ? 'en' : 'vi'">
          {{ guideLang === 'vi' ? 'EN' : 'VI' }}
        </button>
      </div>

      <!-- How to Play -->
      <div v-if="activeTab === 'play'" class="guide-content">
        <table class="guide-table">
          <thead>
            <tr>
              <th>{{ guideLang === 'vi' ? 'Hành động' : 'Action' }}</th>
              <th>{{ guideLang === 'vi' ? 'Bàn phím' : 'Keyboard' }}</th>
              <th>{{ guideLang === 'vi' ? 'Cảm ứng' : 'Touch' }}</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>{{ guideLang === 'vi' ? 'Di chuyển trái/phải' : 'Move left/right' }}</td>
              <td>Arrow / A, D</td>
              <td>{{ guideLang === 'vi' ? 'Nút trái/phải' : 'Left/Right buttons' }}</td>
            </tr>
            <tr>
              <td>{{ guideLang === 'vi' ? 'Nhảy' : 'Jump' }}</td>
              <td>Space / W / Arrow Up</td>
              <td>{{ guideLang === 'vi' ? 'Nút JUMP' : 'JUMP button' }}</td>
            </tr>
            <tr>
              <td>{{ guideLang === 'vi' ? 'Nhảy cao hơn' : 'Jump higher' }}</td>
              <td>{{ guideLang === 'vi' ? 'Giữ phím nhảy' : 'Hold jump key' }}</td>
              <td>{{ guideLang === 'vi' ? 'Giữ nút JUMP' : 'Hold JUMP' }}</td>
            </tr>
            <tr>
              <td>{{ guideLang === 'vi' ? 'Bật/tắt âm thanh' : 'Toggle sound' }}</td>
              <td>M</td>
              <td>{{ guideLang === 'vi' ? 'Nút SOUND' : 'SOUND button' }}</td>
            </tr>
          </tbody>
        </table>

        <table class="guide-table">
          <thead>
            <tr>
              <th>{{ guideLang === 'vi' ? 'Vật phẩm' : 'Item' }}</th>
              <th>{{ guideLang === 'vi' ? 'Hiệu ứng' : 'Effect' }}</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>{{ guideLang === 'vi' ? 'Khối ? (vàng)' : '? Block (yellow)' }}</td>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Nhảy đụng đầu để mở — nhận xu, nấm hoặc ngôi sao'
                    : 'Hit from below — get coins, mushroom or star'
                }}
              </td>
            </tr>
            <tr>
              <td>{{ guideLang === 'vi' ? 'Nấm (đỏ)' : 'Mushroom (red)' }}</td>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Mario to lên, chịu được 1 lần trúng đòn'
                    : 'Mario grows big, can take 1 extra hit'
                }}
              </td>
            </tr>
            <tr>
              <td>{{ guideLang === 'vi' ? 'Ngôi sao' : 'Star' }}</td>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Bất tử tạm thời — tiêu diệt kẻ địch khi chạm vào'
                    : 'Temporary invincibility — destroy enemies on contact'
                }}
              </td>
            </tr>
            <tr>
              <td>{{ guideLang === 'vi' ? 'Xu vàng' : 'Gold coin' }}</td>
              <td>
                {{
                  guideLang === 'vi'
                    ? '+100 điểm, thu 100 xu = +1 mạng'
                    : '+100 points, collect 100 = +1 life'
                }}
              </td>
            </tr>
            <tr>
              <td>{{ guideLang === 'vi' ? 'Cờ (cuối màn)' : 'Flag (end of level)' }}</td>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Chạm cờ để hoàn thành màn chơi'
                    : 'Touch the flag to clear the level'
                }}
              </td>
            </tr>
          </tbody>
        </table>

        <table class="guide-table">
          <thead>
            <tr>
              <th>{{ guideLang === 'vi' ? 'Kẻ địch' : 'Enemy' }}</th>
              <th>{{ guideLang === 'vi' ? 'Cách tiêu diệt' : 'How to defeat' }}</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>Goomba</td>
              <td>
                {{ guideLang === 'vi' ? 'Nhảy lên đầu để giết' : 'Jump on its head to kill' }}
              </td>
            </tr>
            <tr>
              <td>Koopa</td>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Nhảy lên đầu -> mai rùa -> đá mai rùa để giết kẻ địch khác'
                    : 'Stomp -> shell -> kick shell to kill other enemies'
                }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Boss Fight Guide -->
      <div v-if="activeTab === 'boss'" class="guide-content">
        <table class="guide-table">
          <thead>
            <tr>
              <th>{{ guideLang === 'vi' ? 'Giai đoạn' : 'Phase' }}</th>
              <th>{{ guideLang === 'vi' ? 'Hành vi của Bowser' : 'Bowser behavior' }}</th>
              <th>{{ guideLang === 'vi' ? 'Cách đánh' : 'Strategy' }}</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>{{ guideLang === 'vi' ? 'Giai đoạn 1' : 'Phase 1' }} (HP: 3/3)</td>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Đi bộ, bắn cầu lửa, thỉnh thoảng lao về phía bạn'
                    : 'Walks, shoots fireballs, occasionally charges'
                }}
              </td>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Nhảy lên đầu Bowser để gây sát thương. Khi Bowser lao vào tường sẽ bị choáng — cơ hội tốt nhất để tấn công!'
                    : 'Jump on Bowser\'s head to deal damage. When Bowser charges into a wall, he gets stunned — best chance to attack!'
                }}
              </td>
            </tr>
            <tr>
              <td>{{ guideLang === 'vi' ? 'Giai đoạn 2' : 'Phase 2' }} (HP: 2/3)</td>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Nhanh hơn, thêm chiêu đập đất gây chấn động'
                    : 'Faster, adds ground pound attack'
                }}
              </td>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Tránh xa khi Bowser nhảy lên và đập xuống. Đợi Bowser lao và choáng rồi nhảy lên đầu'
                    : 'Stay away when Bowser jumps and slams. Wait for charge + stun, then stomp'
                }}
              </td>
            </tr>
            <tr>
              <td>{{ guideLang === 'vi' ? 'Giai đoạn 3' : 'Phase 3' }} (HP: 1/3)</td>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Chế độ cuồng nộ — lao nhanh hơn, bắn cầu lửa liên tục'
                    : 'Rage mode — faster charges, rapid fireballs'
                }}
              </td>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Cẩn thận hơn! Bowser lao nhanh và bắn nhiều hơn. Kiên nhẫn đợi cơ hội và nhảy lên đầu lần cuối'
                    : 'Be careful! Bowser charges faster and shoots more. Be patient, wait for opening, land final stomp'
                }}
              </td>
            </tr>
          </tbody>
        </table>

        <table class="guide-table">
          <thead>
            <tr>
              <th>{{ guideLang === 'vi' ? 'Mẹo' : 'Tips' }}</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Nhảy lên đầu Bowser bất cứ lúc nào (khi không bất tử) đều gây sát thương'
                    : 'Stomping Bowser\'s head anytime (when not invincible) deals damage'
                }}
              </td>
            </tr>
            <tr>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Khi Bowser đang bất tử (nhấp nháy), nhảy lên đầu sẽ nảy lên an toàn mà không bị thương'
                    : 'When Bowser is invincible (flashing), stomping bounces you off safely'
                }}
              </td>
            </tr>
            <tr>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Khi Bowser bị choáng (sau khi lao vào tường), bạn có thể đi qua người Bowser mà không bị thương'
                    : 'When Bowser is stunned (after wall charge), you can walk through him safely'
                }}
              </td>
            </tr>
            <tr>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Tránh hố dung nham — 2 hố nham ở 2 bên sàn đấu!'
                    : 'Avoid lava pits — 2 lava gaps on both sides of the arena!'
                }}
              </td>
            </tr>
            <tr>
              <td>
                {{
                  guideLang === 'vi'
                    ? 'Sử dụng các khối gạch ở giữa sàn đấu làm chỗ ẩn nấp tránh cầu lửa'
                    : 'Use the brick platforms in the middle of the arena as cover from fireballs'
                }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Credits -->
    <div class="credits">
      <p>Made by <strong>uydev</strong> | VOL.01 / 2026 | vibe.j2team.org</p>
      <p class="controls-hint">Keys: Arrow/WASD + Space | M = Mute</p>
      <div class="social-links">
        <a href="https://github.com/UyLeQuoc" target="_blank" rel="noopener noreferrer" class="social-link">GitHub</a>
        <span class="social-sep">|</span>
        <a href="https://www.facebook.com/uyledev/" target="_blank" rel="noopener noreferrer" class="social-link">Facebook</a>
      </div>
    </div>
  </div>
</template>

<style scoped>
.mario-game {
  min-height: 100vh;
  min-height: 100dvh;
  background: #1a1a2e;
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: monospace;
  color: #fff;
  padding: 0;
  overflow: hidden;
}

.top-bar {
  width: 100%;
  max-width: 820px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 12px;
  background: rgba(0, 0, 0, 0.5);
}

.back-link {
  color: #aaa;
  text-decoration: none;
  font-size: 13px;
  transition: color 0.2s;
}

.back-link:hover {
  color: #fff;
}

.game-title {
  font-size: 14px;
  font-weight: bold;
  color: #e52521;
}

.mute-btn {
  background: none;
  border: 1px solid #555;
  color: #aaa;
  font-family: monospace;
  font-size: 11px;
  padding: 4px 10px;
  cursor: pointer;
  transition: all 0.2s;
}

.mute-btn:hover {
  border-color: #fff;
  color: #fff;
}

.canvas-wrap {
  width: 100%;
  max-width: 820px;
  aspect-ratio: 800 / 480;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #000;
  border: 2px solid #333;
}

.game-canvas {
  width: 100%;
  height: 100%;
  image-rendering: pixelated;
  image-rendering: crisp-edges;
  display: block;
}

.touch-controls {
  width: 100%;
  max-width: 820px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  user-select: none;
  -webkit-user-select: none;
  touch-action: none;
}

.dpad {
  display: flex;
  gap: 8px;
}

.touch-btn {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  border: 2px solid rgba(255, 255, 255, 0.3);
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  font-size: 20px;
  font-family: monospace;
  font-weight: bold;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  -webkit-tap-highlight-color: transparent;
  touch-action: none;
}

.touch-btn:active {
  background: rgba(255, 255, 255, 0.3);
  border-color: rgba(255, 255, 255, 0.6);
}

.jump-btn {
  width: 80px;
  height: 80px;
  font-size: 14px;
  background: rgba(229, 37, 33, 0.3);
  border-color: rgba(229, 37, 33, 0.5);
}

.jump-btn:active {
  background: rgba(229, 37, 33, 0.6);
}

/* Guide Section */
.guide-section {
  width: 100%;
  max-width: 820px;
  margin-top: 12px;
  border: 1px solid #333;
  background: rgba(0, 0, 0, 0.4);
}

.guide-tabs {
  display: flex;
  border-bottom: 1px solid #333;
}

.tab-btn {
  flex: 1;
  padding: 10px 16px;
  background: none;
  border: none;
  color: #888;
  font-family: monospace;
  font-size: 13px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s;
  border-bottom: 2px solid transparent;
}

.tab-btn:hover {
  color: #ccc;
}

.tab-btn.active {
  color: #e52521;
  border-bottom-color: #e52521;
}

.lang-toggle {
  padding: 10px 16px;
  background: none;
  border: none;
  border-left: 1px solid #333;
  color: #ffd700;
  font-family: monospace;
  font-size: 12px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s;
  min-width: 48px;
}

.lang-toggle:hover {
  color: #fff;
  background: rgba(255, 255, 255, 0.05);
}

.guide-content {
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.guide-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 12px;
}

.guide-table th {
  text-align: left;
  padding: 6px 10px;
  background: rgba(229, 37, 33, 0.15);
  color: #e52521;
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  border-bottom: 1px solid #333;
}

.guide-table td {
  padding: 6px 10px;
  color: #ccc;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  line-height: 1.5;
}

.guide-table tr:last-child td {
  border-bottom: none;
}

.guide-table tr:hover td {
  background: rgba(255, 255, 255, 0.03);
}

.credits {
  padding: 12px;
  text-align: center;
  font-size: 12px;
  color: #666;
}

.controls-hint {
  margin-top: 4px;
  font-size: 11px;
  color: #444;
}

.social-links {
  margin-top: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.social-link {
  color: #888;
  text-decoration: none;
  font-size: 12px;
  transition: color 0.2s;
}

.social-link:hover {
  color: #e52521;
}

.social-sep {
  color: #444;
  font-size: 12px;
}

@media (max-width: 480px) {
  .top-bar {
    padding: 6px 8px;
  }

  .game-title {
    font-size: 12px;
  }

  .touch-btn {
    width: 56px;
    height: 56px;
    font-size: 18px;
  }

  .jump-btn {
    width: 70px;
    height: 70px;
  }
}
</style>
