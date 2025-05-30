<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>XO Game - Sleek UI</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #1f1c2c, #928dab);
      color: #fff;
      text-align: center;
    }

    h1 {
      margin-top: 30px;
      font-size: 2.5em;
    }

    .controls {
      margin-top: 20px;
    }

    .controls label {
      margin: 0 10px;
    }

    .controls select, .controls button {
      padding: 10px 15px;
      font-size: 16px;
      margin: 10px 5px;
      border: none;
      border-radius: 10px;
      background-color: #2c2c3e;
      color: white;
      box-shadow: inset -2px -2px 5px #3c3c5c, inset 2px 2px 5px #1c1c2c;
      transition: all 0.3s ease;
    }

    .controls button:hover, select:hover {
      background-color: #3d3d5f;
      cursor: pointer;
    }

    .board {
      display: grid;
      grid-template-columns: repeat(3, 100px);
      gap: 12px;
      justify-content: center;
      margin: 30px auto;
      padding: 20px;
      background: #2c2c3e;
      border-radius: 20px;
      box-shadow: -5px -5px 15px #3c3c5c, 5px 5px 15px #1c1c2c;
    }

    .cell {
      width: 100px;
      height: 100px;
      background: #1e1e2e;
      border-radius: 15px;
      font-size: 2.5em;
      color: #fff;
      display: flex;
      align-items: center;
      justify-content: center;
      box-shadow: -4px -4px 10px #3a3a5a, 4px 4px 10px #1a1a2a;
      cursor: pointer;
      transition: transform 0.2s ease;
    }

    .cell:hover {
      transform: scale(1.05);
    }

    #status {
      font-size: 1.3em;
      margin-top: 20px;
    }

    .restart {
      margin-top: 15px;
      padding: 12px 24px;
      font-size: 16px;
      background-color: #3d3d5f;
      color: white;
      border: none;
      border-radius: 12px;
      box-shadow: -3px -3px 10px #4d4d7a, 3px 3px 10px #1a1a2a;
      cursor: pointer;
    }

    .restart:hover {
      background-color: #505075;
    }
  </style>
</head>
<body>
  <h1>XO Game</h1>

  <div class="controls">
    <label for="playerSelect">Play as:</label>
    <select id="playerSelect">
      <option value="X">X</option>
      <option value="O">O</option>
    </select>

    <label for="difficultySelect">Difficulty:</label>
    <select id="difficultySelect">
      <option value="easy">Easy</option>
      <option value="medium">Medium</option>
      <option value="hard">Hard</option>
    </select>

    <button onclick="startGame()">Start Game</button>
  </div>

  <div class="board" id="board"></div>
  <p id="status">Choose your settings and click Start</p>
  <button class="restart" onclick="startGame()">Restart Game</button>

  <script>
    const boardElement = document.getElementById('board');
    const statusText = document.getElementById('status');
    const playerSelect = document.getElementById('playerSelect');
    const difficultySelect = document.getElementById('difficultySelect');

    let player = "X";
    let bot = "O";
    let difficulty = "easy";
    let gameState = ["", "", "", "", "", "", "", "", ""];
    let gameActive = true;
    const winCombos = [
      [0,1,2], [3,4,5], [6,7,8],
      [0,3,6], [1,4,7], [2,5,8],
      [0,4,8], [2,4,6]
    ];

    function startGame() {
      player = playerSelect.value;
      bot = player === "X" ? "O" : "X";
      difficulty = difficultySelect.value;
      gameState = ["", "", "", "", "", "", "", "", ""];
      gameActive = true;
      drawBoard();
      statusText.textContent = `Your Turn (${player})`;
      if (bot === "X") setTimeout(botMove, 500);
    }

    function drawBoard() {
      boardElement.innerHTML = "";
      for (let i = 0; i < 9; i++) {
        const cell = document.createElement("div");
        cell.classList.add("cell");
        cell.setAttribute("data-index", i);
        cell.textContent = gameState[i];
        cell.addEventListener("click", handleCellClick);
        boardElement.appendChild(cell);
      }
    }

    function handleCellClick(e) {
      const index = e.target.getAttribute("data-index");
      if (!gameState[index] && gameActive) {
        makeMove(index, player);
        if (gameActive) setTimeout(botMove, 300);
      }
    }

    function makeMove(index, currentPlayer) {
      gameState[index] = currentPlayer;
      drawBoard();
      if (checkWin(currentPlayer)) {
        statusText.textContent = `Player ${currentPlayer} Wins!`;
        gameActive = false;
      } else if (!gameState.includes("")) {
        statusText.textContent = "It's a Draw!";
        gameActive = false;
      } else {
        statusText.textContent = `Turn: ${currentPlayer === player ? bot : player}`;
      }
    }

    function botMove() {
      if (!gameActive) return;

      let move;
      if (difficulty === "easy") {
        move = getRandomMove();
      } else if (difficulty === "medium") {
        move = Math.random() < 0.5 ? getBestMove() : getRandomMove();
      } else {
        move = getBestMove();
      }

      if (move !== undefined) {
        makeMove(move, bot);
      }
    }

    function getRandomMove() {
      const available = gameState
        .map((val, idx) => val === "" ? idx : null)
        .filter(v => v !== null);
      return available[Math.floor(Math.random() * available.length)];
    }

    function getBestMove() {
      return minimax(gameState, bot).index;
    }

    function minimax(board, currentPlayer) {
      const availSpots = board
        .map((val, idx) => val === "" ? idx : null)
        .filter(v => v !== null);

      if (checkWin(player, board)) return { score: -10 };
      if (checkWin(bot, board)) return { score: 10 };
      if (availSpots.length === 0) return { score: 0 };

      const moves = [];

      for (let i = 0; i < availSpots.length; i++) {
        const move = {};
        move.index = availSpots[i];
        board[availSpots[i]] = currentPlayer;

        const result = minimax(board, currentPlayer === bot ? player : bot);
        move.score = result.score;

        board[availSpots[i]] = "";
        moves.push(move);
      }

      let bestMove;
      if (currentPlayer === bot) {
        let bestScore = -Infinity;
        moves.forEach((m, i) => {
          if (m.score > bestScore) {
            bestScore = m.score;
            bestMove = i;
          }
        });
      } else {
        let bestScore = Infinity;
        moves.forEach((m, i) => {
          if (m.score < bestScore) {
            bestScore = m.score;
            bestMove = i;
          }
        });
      }

      return moves[bestMove];
    }

    function checkWin(player, board = gameState) {
      return winCombos.some(combo =>
        combo.every(index => board[index] === player)
      );
    }

    startGame(); // Initialize on load
  </script>
</body>
</html>
