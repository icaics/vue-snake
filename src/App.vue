<script setup>
import {
  computed,
  watch,
  ref,
  reactive,
  onMounted,
  onBeforeUnmount,
} from "vue"

// const props = defineProps(["foo"])
// const emit = defineEmits(["change", "delete"])
// Data
const LEFT = -1
const RIGHT = 1
const UP = -10
const DOWN = 10
const STAGE_W = 240
const NODE_W = 10
const SNAKE_X = 120
const SNAKE_Y = 120
const SNAKE_LENGTH = 5
const SNAKE_SPEED = 300
const LEVEL_UP_MARGIN = 20
const FOOD_CREATE_FORBIDDEN_W = 30
const START_GAME = 'PRESS SPACE'
const GAME_OVER = 'GAME OVER!'
//
const gameArea = ref(null)
let titleMessage = ref(START_GAME)
let isAlive = ref(false)
//
let timer = ref(null)
let food = reactive({
  x: 0,
  y: 0,
})
//
const snake = reactive({
  x: 0,
  y: 0,
  length: 0,
  direction: 0,
  steps: [],
  speed: 0,
});
// Function ---------------------------------------------------
onMounted(() => {
  gameArea.value.focus()
  initGame()
});
function initGame() {
  initData()
  initDirection()
}
function initData() {
  isAlive.value = false
  snake.x = SNAKE_X
  snake.y = SNAKE_Y
  snake.length = SNAKE_LENGTH
  snake.direction = 0
  snake.steps = []
  snake.speed = SNAKE_SPEED
  food.x = -1
  food.y = -1
}
function initDirection() {
  const arr = [-1, 1, -10, 10]
  const d = arr[Math.floor(Math.random() * arr.length)]
  snake.direction = d
  for (let i = 0; i < snake.length; i++) {
    snake.steps.unshift(d)
  }
}
function randInt(max) {
  return Math.floor(Math.random() * max)
}
// --------------------------------------------------- Start Game
function startGame() {
  if (!isAlive.value) {
    isAlive.value = true
    createFood()
    startLooper()
  }
}
function startLooper() {
  timer.value = window.setInterval(() => {
    snakeMove()
    detectSnakeCollision()
    detectEat()
  }, snake.speed)
}
//
function snakeMove() {
  //
  snake.steps.unshift(snake.direction)
  if (snake.steps.length > snake.length) {
    for (let i = 0; i < snake.steps.length - snake.length; i++) {
      // 循环弹出多余的步数记录
      snake.steps.pop()
    }
  }
  //
  const step = snake.steps[0]
  if (Math.abs(step) < 5) {
    snake.x += step * NODE_W
    snake.y += 0
  } else {
    snake.x += 0
    snake.y += (step / 10) * NODE_W
  }
}
function snakeControl(direction) {
  if (Math.abs(direction) === Math.abs(snake.steps[0])) {
    return
  }
  snake.direction = direction
}
function createFood() {
  let x = randInt(STAGE_W / 10) * 10
  let y = randInt(STAGE_W / 10) * 10
  // Check head arround
  if ((snake.x - FOOD_CREATE_FORBIDDEN_W < x && x < snake.x + FOOD_CREATE_FORBIDDEN_W) ||
    (snake.y - FOOD_CREATE_FORBIDDEN_W < y && y < snake.y + FOOD_CREATE_FORBIDDEN_W)) {
    createFood()
  }
  // Check body
  for (let n = 0; n < nodes.value.length; n++) {
    if (x === nodes.value[n].x || y === nodes.value[n].y) {
      createFood()
    }
  }
  food.x = x
  food.y = y
}
function detectSnakeCollision() {
  for (let n = 0; n < nodes.value.length; n++) {
    if (n === 0) {
      continue
    }
    if (snake.x === nodes.value[n].x && snake.y === nodes.value[n].y) {
      gameOver()
    }
  }
}
function detectEat() {
  if (snake.x === food.x && snake.y === food.y) {
    levelUp()
    snake.length++
    snake.steps.push(snake.direction)
    createFood()
  }
}
function levelUp() {
  window.clearInterval(timer.value)
  snake.speed -= LEVEL_UP_MARGIN
  startLooper()
}
function gameOver() {
  titleMessage.value = GAME_OVER
  window.clearInterval(timer.value)
  initGame()
}
onBeforeUnmount(() => {
  window.clearInterval(timer.value)
});
// Computed ---------------------------------------------------
const nodes = computed(() => {
  const result = []
  for (let s = 0; s < snake.length; s++) {
    if (s === 0) {
      result.push({
        x: snake.x,
        y: snake.y,
      })
      continue
    }
    const step = snake.steps[s - 1]
    const lastNode = result[s - 1]
    let x = 0
    let y = 0
    if (Math.abs(step) < 5) {
      x = lastNode.x + -step * NODE_W
      y = lastNode.y
    } else {
      x = lastNode.x
      y = lastNode.y + (-step / 10) * NODE_W
    }
    result.push({
      x: x,
      y: y,
    })
  }
  return result
});
// Watch ---------------------------------------------------
watch(
  () => [snake.x, snake.y],
  ([x, y]) => {
    if (x < 0 || x > STAGE_W - NODE_W || y < 0 || y > STAGE_W - NODE_W) {
      gameOver()
    }
  }
)
</script>

<style scoped>
body {
  overflow: hidden;
}

html {
  overflow: hidden;
  width: 100%;
  height: 100%;
}

#titleMessage {
  position: absolute;
  top: 90px;
  width: 240px;
  text-align: center;
}

#startGame {
  position: absolute;
  top: 260px;
  width: 240px;
  height: 40px;
  color: black;
  background-color: lightskyblue;
  border: 1px solid;
  border-color: grey;
}

#controllers {
  position: absolute;
  margin-top: 14px;
  top: 300px;
  height: 40px;
}

#controllers button {
  margin-right: 13.33px;
  width: 50px;
  height: 40px;
  color: black;
  background-color: lightskyblue;
  border: 1px solid;
  border-color: grey;
}

#gameArea {
  background-color: lightskyblue;
  width: 240px;
  height: 240px;
  position: absolute;
  outline: none;
  border: 1px solid;
  border-color: grey;
}

#snakeNode {
  background-color: blue;
  top: 0px;
  left: 0px;
  width: 10px;
  height: 10px;
  position: absolute;
}

#food {
  background-color: lightcoral;
  top: 0px;
  left: 0px;
  width: 10px;
  height: 10px;
  position: absolute;
}
</style>

<template>
  <div id="gameArea" tabIndex="-1" ref="gameArea" @keydown.space="startGame" @keydown.h="snakeControl(LEFT)"
    @keydown.l="snakeControl(RIGHT)" @keydown.k="snakeControl(UP)" @keydown.j="snakeControl(DOWN)">
    <div id="food" v-show="food.x != -1" :style="{left:food.x + 'px', top:food.y + 'px'}">
    </div>
    <div id="snakeNode" v-for="n in nodes" v-show="isAlive && snake.x >= 0 && snake.y >= 0" key="n.index"
      :style="{ left: n.x + 'px', top: n.y + 'px' }">
    </div>
  </div>
  <h1 id="titleMessage" v-show="!isAlive">{{titleMessage}}</h1>
  <div>
    <button id="startGame" @click="startGame">Start Game</button>
    <br />
    <div id="controllers">
      <button class="controller" @click="snakeControl(LEFT)">← H</button>
      <button class="controller" @click="snakeControl(DOWN)">↓ J</button>
      <button class="controller" @click="snakeControl(UP)">↑ K</button>
      <button class="controller" @click="snakeControl(RIGHT)">→ L</button>
    </div>
  </div>
  <div v-if="false" style="position: absolute; top: 360px">
    <h3>snake:</h3>
    <h5>{{ snake }}</h5>
    <h3>steps:</h3>
    <h5>{{ snake.steps }}</h5>
    <h3>nodes:</h3>
    <h5>{{ nodes }}</h5>
  </div>
</template>
