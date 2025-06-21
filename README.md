```html name=ping_pong_legends_mobile.html
<!DOCTYPE html>
<html>
<head>
  <title>Ping Pong Legends (Mobile)</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <style>
    html, body { margin: 0; padding: 0; overflow: hidden; background: #000; }
    canvas { display: block; margin: 0 auto; background: #000; touch-action: none; }
  </style>
</head>
<body>
<canvas id="game"></canvas>
<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let W = window.innerWidth, H = window.innerHeight;
canvas.width = W;
canvas.height = H;

// Responsive scaling factors
function resizeCanvas() {
    W = window.innerWidth;
    H = window.innerHeight;
    canvas.width = W;
    canvas.height = H;
}
window.addEventListener("resize", resizeCanvas);

function percX(p) { return W * p; }
function percY(p) { return H * p; }

// Game objects
const PADDLE_W = percX(0.03), PADDLE_H = percY(0.18);
let leftPaddle = { x: percX(0.05), y: percY(0.41), w: PADDLE_W, h: PADDLE_H, vy: 0 };
let rightPaddle = { x: percX(0.92), y: percY(0.41), w: PADDLE_W, h: PADDLE_H, vy: 0 };
let ball = { x: W/2, y: H/2, r: percX(0.018), vx: percX(0.008), vy: percY(0.008) };
let leftScore = 0, rightScore = 0;

let powerUps = [];
const POWERUP_SIZE = percX(0.04);
const POWERUP_TYPES = ['enlarge', 'shrink', 'speed', 'multi'];

// Touch tracking
let touchLeft = null, touchRight = null;

function resetBall(dir=1) {
    ball.x = W/2;
    ball.y = H/2;
    ball.vx = percX(0.008) * dir;
    ball.vy = percY(0.008) * (Math.random() > 0.5 ? 1 : -1);
}

function drawRect(obj, color="#fff") {
    ctx.fillStyle = color;
    ctx.fillRect(obj.x, obj.y, obj.w, obj.h);
}
function drawCircle(obj, color="#fff") {
    ctx.beginPath();
    ctx.arc(obj.x, obj.y, obj.r, 0, 2 * Math.PI);
    ctx.fillStyle = color;
    ctx.fill();
}
function drawText(txt, x, y, size, color="#fff") {
    ctx.fillStyle = color;
    ctx.font = `${size}px Arial`;
    ctx.textAlign = "center";
    ctx.fillText(txt, x, y);
}

function spawnPowerUp() {
    if (powerUps.length < 1 && Math.random() < 0.005) {
        powerUps.push({
            x: percX(0.2) + Math.random()*percX(0.6),
            y: percY(0.15) + Math.random()*percY(0.7),
            w: POWERUP_SIZE, h: POWERUP_SIZE,
            type: POWERUP_TYPES[Math.floor(Math.random()*POWERUP_TYPES.length)],
            active: true
        });
    }
}

function applyPowerUp(paddle, type, isLeft) {
    switch(type) {
        case 'enlarge':
            paddle.h *= 1.5;
            setTimeout(()=>{ paddle.h /= 1.5; }, 4000);
            break;
        case 'shrink':
            paddle.h *= 0.6;
            setTimeout(()=>{ paddle.h /= 0.6; }, 4000);
            break;
        case 'speed':
            // Not implemented: could increase ball speed
            ball.vx *= 1.4; ball.vy *= 1.4;
            setTimeout(()=>{ ball.vx /= 1.4; ball.vy /= 1.4; }, 4000);
            break;
        case 'multi':
            // Not implemented: add second ball
            break;
    }
}

function gameLoop() {
    ctx.clearRect(0,0,W,H);

    // Draw paddles, ball, scores
    drawRect(leftPaddle);
    drawRect(rightPaddle);
    drawCircle(ball);
    drawText(leftScore, percX(0.25), percY(0.12), percX(0.09));
    drawText(rightScore, percX(0.75), percY(0.12), percX(0.09));

    // Move paddles
    leftPaddle.y += leftPaddle.vy;
    rightPaddle.y += rightPaddle.vy;
    leftPaddle.y = Math.max(0, Math.min(H-leftPaddle.h, leftPaddle.y));
    rightPaddle.y = Math.max(0, Math.min(H-rightPaddle.h, rightPaddle.y));

    // Move ball
    ball.x += ball.vx;
    ball.y += ball.vy;

    // Wall collision
    if (ball.y-ball.r < 0 || ball.y+ball.r > H) {
        ball.vy *= -1;
    }

    // Paddle collision
    if (
        ball.x-ball.r < leftPaddle.x+leftPaddle.w &&
        ball.y > leftPaddle.y && ball.y < leftPaddle.y+leftPaddle.h
    ) {
        ball.vx = Math.abs(ball.vx);
    }
    if (
        ball.x+ball.r > rightPaddle.x &&
        ball.y > rightPaddle.y && ball.y < rightPaddle.y+rightPaddle.h
    ) {
        ball.vx = -Math.abs(ball.vx);
    }

    // Score
    if (ball.x < 0) { rightScore++; resetBall(1); }
    if (ball.x > W) { leftScore++; resetBall(-1); }

    // Powerups
    spawnPowerUp();
    powerUps.forEach((pu, i) => {
        let color = "#0f0";
        if (pu.type === 'shrink') color = '#f00';
        if (pu.type === 'speed') color = '#00f';
        if (pu.type === 'multi') color = '#ff0';
        drawRect(pu, color);

        // Collision
        if (
            pu.active && ball.x > pu.x && ball.x < pu.x+pu.w &&
            ball.y > pu.y && ball.y < pu.y+pu.h
        ) {
            pu.active = false;
            // Apply to side ball is moving towards
            if (ball.vx > 0) applyPowerUp(rightPaddle, pu.type, false);
            else applyPowerUp(leftPaddle, pu.type, true);
        }
    });
    powerUps = powerUps.filter(pu => pu.active);

    requestAnimationFrame(gameLoop);
}

// Touch controls
canvas.addEventListener("touchstart", function(e) {
    for (let t of e.touches) {
        // Left or right half?
        if (t.clientX < W/2) touchLeft = t.identifier;
        else touchRight = t.identifier;
    }
});
canvas.addEventListener("touchmove", function(e) {
    for (let t of e.touches) {
        if (t.identifier === touchLeft) {
            // Move left paddle
            leftPaddle.y = Math.min(H-leftPaddle.h, Math.max(0, t.clientY-leftPaddle.h/2));
        }
        if (t.identifier === touchRight) {
            rightPaddle.y = Math.min(H-rightPaddle.h, Math.max(0, t.clientY-rightPaddle.h/2));
        }
    }
});
canvas.addEventListener("touchend", function(e) {
    for (let t of e.changedTouches) {
        if (t.identifier === touchLeft) touchLeft = null;
        if (t.identifier === touchRight) touchRight = null;
    }
});

resizeCanvas();
gameLoop();
</script>
</body>
</html>
```
