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
let placarIA = 0;
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

// Eventos de teclado com prevenção do scroll da tela nas setas e espaço
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

// Timer do jogo
const cronometro = setInterval(() => {
    if (tempoRestante > 0) {
        tempoRestante--;
    } else {
        jogoAtivo = false;
        clearInterval(cronometro);
    }
}, 1000);

function chutar() {
    const dx = bola.meusite
