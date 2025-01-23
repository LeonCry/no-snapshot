<script setup lang="ts">
const canvas = ref<HTMLCanvasElement | null>(null);
const gameOver = ref(false);
const gameStarted = ref(false);
const intervalId = ref<ReturnType<typeof setInterval> | null>(null);
const score = ref(0);

const ctx = ref<CanvasRenderingContext2D | null>(null);
const snake = ref([
  { x: 100, y: 100 },
  { x: 80, y: 100 },
  { x: 60, y: 100 },
]);
const food = ref({ x: 200, y: 200 });
const direction = ref({ x: 20, y: 0 });
const speed = 100;

function drawSnake() {
  if (!ctx.value)
    return;
  ctx.value.fillStyle = 'green';
  snake.value.forEach((segment) => {
    ctx.value!.fillRect(segment.x, segment.y, 20, 20);
  });
}

function drawFood() {
  if (!ctx.value)
    return;
  ctx.value.fillStyle = 'red';
  ctx.value.fillRect(food.value.x, food.value.y, 20, 20);
}

function moveSnake() {
  const head = { x: snake.value[0].x + direction.value.x, y: snake.value[0].y + direction.value.y };
  snake.value.unshift(head);

  if (head.x === food.value.x && head.y === food.value.y) {
    score.value += 10;
    food.value = {
      x: Math.floor(Math.random() * 20) * 20,
      y: Math.floor(Math.random() * 20) * 20,
    };
  }
  else {
    snake.value.pop();
  }
}

function checkCollision() {
  const head = snake.value[0];
  if (
    head.x < 0
    || head.x >= 400
    || head.y < 0
    || head.y >= 400
    || snake.value.slice(1).some(segment => segment.x === head.x && segment.y === head.y)
  ) {
    gameOver.value = true;
  }
}

function gameLoop() {
  if (!ctx.value)
    return;
  ctx.value.clearRect(0, 0, 400, 400);
  moveSnake();
  checkCollision();
  drawSnake();
  drawFood();
}

function startGame() {
  if (!canvas.value)
    return;
  ctx.value = canvas.value.getContext('2d');
  gameStarted.value = true;
  intervalId.value = setInterval(gameLoop, speed);
}

function restartGame() {
  if (intervalId.value) {
    clearInterval(intervalId.value);
    intervalId.value = null;
  }

  score.value = 0;
  snake.value = [
    { x: 100, y: 100 },
    { x: 80, y: 100 },
    { x: 60, y: 100 },
  ];
  food.value = { x: 200, y: 200 };
  direction.value = { x: 20, y: 0 };
  gameOver.value = false;
  startGame();
}

function handleKey(e: KeyboardEvent) {
  switch (e.key) {
    case 'ArrowUp':
      if (direction.value.y === 0)
        direction.value = { x: 0, y: -20 };
      break;
    case 'ArrowDown':
      if (direction.value.y === 0)
        direction.value = { x: 0, y: 20 };
      break;
    case 'ArrowLeft':
      if (direction.value.x === 0)
        direction.value = { x: -20, y: 0 };
      break;
    case 'ArrowRight':
      if (direction.value.x === 0)
        direction.value = { x: 20, y: 0 };
      break;
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKey);
});

onUnmounted(() => {
  if (intervalId.value)
    clearInterval(intervalId.value);
  window.removeEventListener('keydown', handleKey);
});
</script>

<template>
  <section class="w-full h-full flex flex-col justify-center items-center gap-10">
    <h1 class="text-2xl font-bold text-red-500">
      该项目未配置链接/链接无效/无预览功能
    </h1>
    <div class="game-container">
      <div
        v-if="gameStarted && !gameOver"
        class="score-display"
      >
        得分: {{ score }}
      </div>
      <canvas
        ref="canvas"
        width="400"
        height="400"
      />
      <div
        v-if="!gameStarted"
        class="start-game"
      >
        <h2>贪吃蛇</h2>
        <button
          class="mt-5 p-2 bg-blue-100 rounded-md hover:bg-blue-200"
          @click="startGame"
        >
          开始游戏
        </button>
      </div>
      <div
        v-if="gameOver"
        class="game-over"
      >
        <h2>游戏结束</h2>
        <p>最终得分: {{ score }}</p>
        <button
          class="mt-5 p-2 bg-red-100 rounded-md hover:bg-red-200"
          @click="restartGame"
        >
          重新开始
        </button>
      </div>
    </div>
  </section>
</template>

<style>
.game-container {
  position: relative;
  width: 400px;
  height: 400px;
  border: 2px solid #000;
}
.game-over {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: rgba(255, 255, 255, 0.8);
  padding: 20px;
  border-radius: 10px;
  text-align: center;
}
.start-game {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: rgba(255, 255, 255, 0.8);
  padding: 20px;
  border-radius: 10px;
  text-align: center;
}
.score-display {
  position: absolute;
  top: 10px;
  right: 10px;
  background-color: rgba(255, 255, 255, 0.8);
  padding: 5px 10px;
  border-radius: 5px;
  font-size: 18px;
  font-weight: bold;
}
</style>
