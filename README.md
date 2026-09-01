<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Jogo de Futebol Interativo</title>

<style>
    body {
        margin: 0;
        background: #0d1117;
        color: white;
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        text-align: center;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        min-height: 100vh;
    }

    h1 {
        margin: 10px 0 5px 0;
        font-size: 2.2rem;
        text-shadow: 0 2px 4px rgba(0,0,0,0.5);
    }

    p {
        font-size: 15px;
        color: #a3b8cc;
        margin-bottom: 15px;
    }

    canvas {
        background: #2e8b57;
        border: 4px solid #ffffff;
        border-radius: 8px;
        box-shadow: 0 10px 25px rgba(0, 0, 0, 0.7);
        max-width: 95%;
    }
</style>
</head>

<body>

<h1>⚽ Futebol Show</h1>
<p><b>WASD / Setas</b>: Mover | <b>Espaço</b>: Chutar Forte | <b>Empurre a bola</b> para conduzir</p>

<canvas id="campo" width="900" height="500"></canvas>

<script>
const canvas = document.getElementById("campo");
const ctx = canvas.getContext("2d");

// Estado do Jogo
let placarJogador = 0;
let tempoRestante = 60; // Segundos
let jogoAtivo = true;

const jogador = {
    x: 250,
    y: 250,
    tamanho: 18,
    velocidade: 4.5,
    cor: "#0088ff"
};

const goleiro = {
    x: 840,
    y: 250,
    tamanho: 18,
    velocidade: 2.8,
    cor: "#ff3333"
};

const bola = {
    x: 450,
    y: 250,
    raio: 9,
    vx: 0,
    vy: 0,
    atrito: 0.981
};

const gol = {
    x: 860,
    y: 170,
    largura: 30,
    altura: 160
};

let teclas = {};

// Eventos de teclado
document.addEventListener("keydown", e => {
    teclas[e.key.toLowerCase()] = true;
    if (["Space", "ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight"].includes(e.code)) {
        e.preventDefault();
    }
    if (e.code === "Space" && jogoAtivo) {
        chutar();
    }
});

document.addEventListener("keyup", e => {
    teclas[e.key.toLowerCase()] = false;
});

// Temporizador de partida
const cronometro = setInterval(() => {
    if (tempoRestante > 0) {
        tempoRestante--;
    } else {
        jogoAtivo = false;
        clearInterval(cronometro);
    }
}, 1000);

function chutar() {
    const dx = bola.x - jogador.x;
    const dy = bola.y - jogador.y;
    const distancia = Math.hypot(dx, dy);

    if (distancia < jogador.tamanho + bola.raio + 15) {
        const angulo = Math.atan2(dy, dx);
        const força = 13;
        bola.vx = Math.cos(angulo) * força;
        bola.vy = Math.sin(angulo) * força;
    }
}

function moverJogador() {
    if (!jogoAtivo) return;

    if (teclas["arrowup"] || teclas["w"]) jogador.y -= jogador.velocidade;
    if (teclas["arrowdown"] || teclas["s"]) jogador.y += jogador.velocidade;
    if (teclas["arrowleft"] || teclas["a"]) jogador.x -= jogador.velocidade;
    if (teclas["arrowright"] || teclas["d"]) jogador.x += jogador.velocidade;

    jogador.x = Math.max(jogador.tamanho + 10, Math.min(canvas.width - jogador.tamanho - 10, jogador.x));
    jogador.y = Math.max(jogador.tamanho + 10, Math.min(canvas.height - jogador.tamanho - 10, jogador.y));
}

function moverGoleiro() {
    if (!jogoAtivo) return;

    const centroGolY = gol.y + gol.altura / 2;
    let alvoY = bola.x > canvas.width / 2 ? bola.y : centroGolY;

    alvoY = Math.max(gol.y + goleiro.tamanho, Math.min(gol.y + gol.altura - goleiro.tamanho, alvoY));

    if (goleiro.y < alvoY - 2) goleiro.y += goleiro.velocidade;
    else if (goleiro.y > alvoY + 2) goleiro.y -= goleiro.velocidade;
}

function processarColisoes() {
    resolverColisaoEsferas(jogador, bola);
    resolverColisaoEsferas(goleiro, bola);
}

function resolverColisaoEsferas(p, b) {
    const dx = b.x - p.x;
    const dy = b.y - p.y;
    const distancia = Math.hypot(dx, dy);
    const minDist = p.tamanho + b.raio;

    if (distancia < minDist) {
        const angulo = Math.atan2(dy, dx);
        const overlap = minDist - distancia;

        b.x += Math.cos(angulo) * overlap;
        b.y += Math.sin(angulo) * overlap;

        b.vx += Math.cos(angulo) * 1.5;
        b.vy += Math.sin(angulo) * 1.5;
    }
}

function moverBola() {
    bola.x += bola.vx;
    bola.y += bola.vy;

    bola.vx *= bola.atrito;
    bola.vy *= bola.atrito;

    if (bola.y - bola.raio < 10) {
        bola.y = 10 + bola.raio;
        bola.vy *= -0.8;
    }
    if (bola.y + bola.raio > canvas.height - 10) {
        bola.y = canvas.height - 10 - bola.raio;
        bola.vy *= -0.8;
    }

    if (bola.x - bola.raio < 10) {
        bola.x = 10 + bola.raio;
        bola.vx *= -0.8;
    }

    if (bola.x + bola.raio > canvas.width - 10) {
        const dentroDoGolY = bola.y > gol.y && bola.y < gol.y + gol.altura;

        if (dentroDoGolY) {
            placarJogador++;
            resetarPosicoes();
        } else {
            bola.x = canvas.width - 10 - bola.raio;
            bola.vx *= -0.8;
        }
    }
}

function resetarPosicoes() {
    bola.x = canvas.width / 2;
    bola.y = canvas.height / 2;
    bola.vx = 0;
    bola.vy = 0;

    jogador.x = 250;
    jogador.y = 250;

    goleiro.x = 840;
    goleiro.y = 250;
}

function desenharCampo() {
    const larguraListra = 60;
    for (let i = 0; i < canvas.width; i += larguraListra) {
        ctx.fillStyle = (i / larguraListra) % 2 === 0 ? "#2e8b57" : "#287a4c";
        ctx.fillRect(i, 0, larguraListra, canvas.height);
    }

    ctx.strokeStyle = "rgba(255, 255, 255, 0.7)";
    ctx.lineWidth = 3;

    ctx.strokeRect(10, 10, canvas.width - 20, canvas.height - 20);

    ctx.beginPath();
    ctx.moveTo(canvas.width / 2, 10);
    ctx.lineTo(canvas.width / 2, canvas.height - 10);
    ctx.stroke();

    ctx.beginPath();
    ctx.arc(canvas.width / 2, canvas.height / 2, 60, 0, Math.PI * 2);
    ctx.stroke();

    ctx.fillStyle = "rgba(255, 255, 255, 0.15)";
    ctx.fillRect(gol.x, gol.y, gol.largura, gol.altura);
    ctx.strokeRect(gol.x, gol.y, gol.largura, gol.altura);

    ctx.lineWidth = 1;
    ctx.strokeStyle = "rgba(255, 255, 255, 0.3)";
    for (let y = gol.y; y <= gol.y + gol.altura; y += 10) {
        ctx.beginPath();
        ctx.moveTo(gol.x, y);
        ctx.lineTo(gol.x + gol.largura, y);
        ctx.stroke();
    }

    ctx.fillStyle = jogador.cor;
    ctx.beginPath();
    ctx.arc(jogador.x, jogador.y, jogador.tamanho, 0, Math.PI * 2);
    ctx.fill();
    ctx.strokeStyle = "white";
    ctx.lineWidth = 2;
    ctx.stroke();

    ctx.fillStyle = goleiro.cor;
    ctx.beginPath();
    ctx.arc(goleiro.x, goleiro.y, goleiro.tamanho, 0, Math.PI * 2);
    ctx.fill();
    ctx.strokeStyle = "white";
    ctx.lineWidth = 2;
    ctx.stroke();

    ctx.fillStyle = "#ffffff";
    ctx.beginPath();
    ctx.arc(bola.x, bola.y, bola.raio, 0, Math.PI * 2);
    ctx.fill();
    ctx.strokeStyle = "#333333";
    ctx.lineWidth = 1;
    ctx.stroke();

    ctx.fillStyle = "white";
    ctx.font = "bold 20px Arial";
    ctx.fillText("GOLS: " + placarJogador, 30, 40);
    ctx.fillText("TEMPO: " + tempoRestante + "s", canvas.width - 160, 40);

    if (!jogoAtivo) {
        ctx.fillStyle = "rgba(0, 0, 0, 0.75)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = "#ffffff";
        ctx.font = "bold 36px Arial";
        ctx.fillText("FIM DE JOGO!", canvas.width / 2 - 120, canvas.height / 2 - 20);
        ctx.font = "22px Arial";
        ctx.fillText("Total de Gols: " + placarJogador, canvas.width / 2 - 80, canvas.height / 2 + 20);
        ctx.font = "16px Arial";
        ctx.fillText("Atualize a página para jogar novamente", canvas.width / 2 - 140, canvas.height / 2 + 60);
    }
}

function loop() {
    moverJogador();
    moverGoleiro();
    processarColisoes();
    moverBola();
    desenharCampo();

    requestAnimationFrame(loop);
}

loop();
</script>

</body>
</html>