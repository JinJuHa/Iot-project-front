<template>
  <div class="about">
    <div class="NUM">
      <span class="MOC">목숨: <i id="life">5</i></span>
      <span class="MOC" style="margin-left: 10px;">점수: <i id="score">0</i></span>
      <span class="MOC" v-if="NUM" style="margin-left: 10px;">이전 점수: <i id="backscore">{{ NUM }}</i></span>
    </div>
    <div>
        <h2>스페이스를 눌러 게임을 시작</h2>
        <button class="logout" @click="Logout">로그아웃</button>
        <canvas id="canvas"></canvas>
        <!-- <img class="BACK" src="../../public/christmas.jpg" /> -->
    </div>
  </div>
</template>

<script>
import { io } from 'socket.io-client'

var soundCatch = {};
soundCatch.data = 0; //네임스페이스 객체를 선언하여 전역 변수로

export default {
  data(){
    return {
      dinoImg : require(`../../public/santa.png`),
      treeImg : require(`../../public/tree.png`),
      NUM: localStorage.getItem('score'),
      socket: null,
      final: ''
    }
  },
async created() {
    this.socket = io(
      process.env.VUE_APP_URL + '/about',
      {
        cors: { origin: '*' }
      }
    )

    this.socket.emit('my message', 'Hello there from Vue.');
     this.socket.on('my broadcast', (data) => {
        console.log(data);
        this.final = data
        console.log(this.final);
        soundCatch.data++; 
    });
  },

 mounted() {
    

let canvas = document.querySelector('#canvas');
let ctx = canvas.getContext('2d'); // context 란 뜻으로 ctx

canvas.width = window.innerWidth - 100;
canvas.height = window.innerHeight - 100;

// let img_two
// function IMG() {
// img_two = new Image (); //이미지 객체 생성
// img_two.src = 'christmas.jpg'; //code.jpg라는 이미지 파일을 로딩 시작
// }

// function LEN() {
//     ctx.drawImage (img_two, 50,50, canvas.width, canvas.height);
// }

// function Main() {
//     LEN()
//     requestAnimationFrame(Main)
// }

// IMG()
// Main()

let dinoImg = new Image();
dinoImg.src = 'santa.png';
let dino = {
    x: 10,
    y: 200,
    width: 40,
    height: 50,
    draw() {
        // ctx.fillStyle = 'green';
        // ctx.fillRect(this.x, this.y, this.width, this.height);
        ctx.drawImage(dinoImg, this.x, this.y);
    }
}

let treeImg = new Image();
treeImg.src = 'tree.png';
class Cactus {
    constructor() {
        this.width = 40 + getRandomInt(-3, 4);
        this.height = 50 + getRandomInt(-3, 4);
        this.x = 500;
        this.y = 250 - this.height;
    }
    draw() {
        // ctx.fillStyle = 'red';
        // ctx.fillRect(this.x, this.y, this.width, this.height);
        ctx.drawImage(treeImg, this.x, this.y, this.width, this.height);
    }
}


let timer = 0;
let cactusArr = [];
let gameState = 0; // 0: end, 1: start
let jumpState = 0; // 0: default, 1: jumping
let jumpTimer = 0;
let animation;
let life = 5;
let score = 0;
let temp = 0; //eslint-disable-line no-unused-vars

function frameAction() {
    animation = requestAnimationFrame(frameAction);
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    timer += 1;
    
    if(timer % 120 == 0){
        let cactus = new Cactus();
        cactusArr.push(cactus);
    }
    cactusArr.forEach((a, i, o)=>{
        if(a.x < 0){
            o.splice(i, 1);
            score += 10;
            document.querySelector('#score').innerHTML = score;
        } else if(collisionDetection(dino, a) < 0){
            o.splice(i, 1);
        }
        
        a.x -= 2;
        a.draw();
    })    

    if(jumpState == 1){
        jumpTimer++; 
        dino.y -= 2;
    }
    if(jumpTimer > 50){
        jumpState = 0;
        jumpTimer = 0;
    }
    if(jumpState == 0){
        if(dino.y < 200){
            dino.y += 2;
        }
    }

    drawLine();
    dino.draw();
}



//점프하는구간
document.addEventListener('keydown', (e)=>{
    if(e.code == 'Space'){
        if(gameState == 0){
            gameState = 1; // 게임실행
            frameAction();
            document.querySelector('h2').style.display = 'none';
            temp = soundCatch.data ;
        } 
        // else if(temp != soundCatch.data){ // 게임실행중일때 스페이스누르면
        //      jumpState = 1;
        //      temp = soundCatch.data ;
        //  }
        }
})

FIRE()

function FIRE(){
        var timmer = setInterval(setTimer, 500);
        setTimeout(stopTimer, 100000);
        function setTimer() {
            while (gameState == 1) {
            if(temp != soundCatch.data){ // 게임실행중일때 스페이스누르면
                    jumpState = 1;
                    temp = soundCatch.data ;
                }
            // location.reload();
            // frameAction();
            break
            }
        }
        function stopTimer() {
        clearInterval(timmer);
        }
}




function collisionDetection(dino, cactus){
    let xValue = cactus.x - ( dino.x + dino.width );
    let yValue = cactus.y - ( dino.y + dino.height );
    if( xValue < 0 && yValue < 0 ){ // 충돌!
        // 충돌 시 실행되는 코드
        life--;
        document.querySelector('#life').innerHTML = life;
        if(life == 0){
            localStorage.setItem("score", score)
            alert('게임오버! 당신의 스코어는' + score + '점 입니다.');
            gameState = 0;
            cancelAnimationFrame(animation);
            location.reload();
        }
        return -1;
    } else {
        return 1;
    }
}

function getRandomInt(min, max) {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min)) + min; //최댓값은 제외, 최솟값은 포함
}

function drawLine(){
    ctx.beginPath();
    ctx.moveTo(0, 250);
    ctx.lineTo(600, 250);
    ctx.stroke();
}
  },
  methods: {
      Logout() {
      localStorage.removeItem('token')
      localStorage.removeItem('score')
      this.$router.push('/')
    }
  }
};
</script>
<style>
.about {
  width: 100%;
}
.BACK {
    position: absolute;
    width: 600px;
    margin: 0px;
    padding: 0px;
    z-index: -1;
}
.MOC {
    font-size: 20px;
    font-weight: bold;
}
.logout {
    margin-top: 10px;
}
/* #canvas {
    position: absolute;
    z-index: -1;
} */
</style>