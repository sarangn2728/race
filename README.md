<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bike Racing Game</title>

<!-- Bootstrap -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">

<style>
body {
    margin: 0;
    background: linear-gradient(#87ceeb, #e0f7fa);
    overflow: hidden;
    font-family: 'Segoe UI', sans-serif;
}

/* Game Container */
#gameContainer {
    position: relative;
    width: 100%;
    max-width: 420px;
    height: 100vh;
    margin: auto;
    overflow: hidden;
    background: #555;
}

/* Road */
#road {
    position: absolute;
    width: 100%;
    height: 200%;
    background: repeating-linear-gradient(
        #444 0px,
        #444 40px,
        #fff 40px,
        #fff 60px
    );
    animation: roadMove 1s linear infinite;
}

@keyframes roadMove {
    from { transform: translateY(-50%); }
    to { transform: translateY(0); }
}

/* Bike */
#bike {
    position: absolute;
    bottom: 60px;
    left: 50%;
    transform: translateX(-50%);
    width: 60px;
    height: 110px;
    background: linear-gradient(red, darkred);
    border-radius: 15px;
    box-shadow: 0 0 15px rgba(0,0,0,0.6);
}

#bike::before {
    content: '';
    position: absolute;
    bottom: -15px;
    left: 5px;
    width: 50px;
    height: 20px;
    background: black;
    border-radius: 50%;
}

/* Obstacle */
.obstacle {
    position: absolute;
    width: 60px;
    height: 60px;
    background: #222;
    border-radius: 10px;
    box-shadow: inset 0 0 10px red;
}

/* HUD */
#hud {
    position: absolute;
    top: 10px;
    width: 100%;
    color: white;
    text-align: center;
    z-index: 10;
}

#controls {
    position: absolute;
    bottom: 10px;
    width: 100%;
    display: flex;
    justify-content: space-around;
}

.control-btn {
    width: 80px;
    height: 80px;
    border-radius: 50%;
    background: rgba(0,0,0,0.6);
    color: white;
    font-size: 30px;
    border: none;
}
</style>
</head>

<body>

<div id="gameContainer">
    <div id="road"></div>

    <div id="hud">
        <h4>Speed: <span id="speed">0</span> km/h</h4>
        <h5>Score: <span id="score">0</span></h5>
    </div>

    <div id="bike"></div>

    <div id="controls">
        <button class="control-btn" id="leftBtn">⬅</button>
        <button class="control-btn" id="rightBtn">➡</button>
    </div>
</div>

<!-- Game Over Modal -->
<div class="modal fade" id="gameOverModal" tabindex="-1">
  <div class="modal-dialog modal-dialog-centered">
    <div class="modal-content text-center">
      <div class="modal-header">
        <h5 class="modal-title">Game Over</h5>
      </div>
      <div class="modal-body">
        <p>Your Score: <span id="finalScore"></span></p>
        <button class="btn btn-danger" onclick="location.reload()">Restart</button>
      </div>
    </div>
  </div>
</div>

<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>

<script>
let bike = document.getElementById('bike');
let gameContainer = document.getElementById('gameContainer');
let speedEl = document.getElementById('speed');
let scoreEl = document.getElementById('score');

let bikeX = gameContainer.offsetWidth / 2 - 30;
let speed = 5;
let score = 0;
let obstacles = [];
let gameRunning = true;

/* Controls */
$('#leftBtn').on('touchstart mousedown', () => bikeX -= 40);
$('#rightBtn').on('touchstart mousedown', () => bikeX += 40);

document.addEventListener('keydown', e => {
    if (e.key === 'ArrowLeft') bikeX -= 30;
    if (e.key === 'ArrowRight') bikeX += 30;
});

/* Create Obstacle */
function createObstacle() {
    let obs = document.createElement('div');
    obs.className = 'obstacle';
    obs.style.left = Math.random() * (gameContainer.offsetWidth - 60) + 'px';
    obs.style.top = '-80px';
    gameContainer.appendChild(obs);
    obstacles.push(obs);
}

/* Collision */
function isColliding(a, b) {
    let r1 = a.getBoundingClientRect();
    let r2 = b.getBoundingClientRect();
    return !(r1.right < r2.left ||
             r1.left > r2.right ||
             r1.bottom < r2.top ||
             r1.top > r2.bottom);
}

/* Game Loop */
function gameLoop() {
    if (!gameRunning) return;

    bikeX = Math.max(0, Math.min(gameContainer.offsetWidth - 60, bikeX));
    bike.style.left = bikeX + 'px';

    obstacles.forEach((obs, index) => {
        obs.style.top = obs.offsetTop + speed + 'px';

        if (isColliding(bike, obs)) {
            gameRunning = false;
            $('#finalScore').text(score);
            new bootstrap.Modal('#gameOverModal').show();
        }

        if (obs.offsetTop > gameContainer.offsetHeight) {
            obs.remove();
            obstacles.splice(index, 1);
            score++;
            speed += 0.2;
        }
    });

    speedEl.textContent = Math.floor(speed * 10);
    scoreEl.textContent = score;

    if (Math.random() < 0.02) createObstacle();

    requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

</body>
</html>
