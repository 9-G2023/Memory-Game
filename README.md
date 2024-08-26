
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Memory Game</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background-color: #f0f0f0;
            margin: 0;
            padding: 0;
        }

        h1 {
            margin-top: 20px;
        }

        #controls {
            margin: 20px;
        }

        #difficulty {
            margin-right: 10px;
        }

        #game-board {
            display: grid;
            gap: 10px;
            justify-content: center;
            margin: 20px auto;
        }

        .card {
            width: 100px;
            height: 100px;
            background-color: #333;
            color: #333;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 24px;
            cursor: pointer;
            border-radius: 8px;
            user-select: none;
            transition: transform 0.5s;
            transform-style: preserve-3d;
        }

        .card.flipped {
            background-color: #fff;
            color: #000;
        }

        .card.flipped .inner {
            transform: rotateY(180deg);
        }

        .card .inner {
            position: relative;
            width: 100%;
            height: 100%;
            transform-style: preserve-3d;
            transition: transform 0.5s;
        }

        .card .front, .card .back {
            position: absolute;
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            backface-visibility: hidden;
        }

        .card .front {
            background-color: #333;
            color: #333;
        }

        .card .back {
            background-color: #fff;
            color: #000;
            transform: rotateY(180deg);
        }

        #info {
            margin-top: 20px;
            font-size: 18px;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1>Memory Game</h1>
    <div id="controls">
        <label for="difficulty">Tingkat Kesulitan:</label>
        <select id="difficulty">
            <option value="4x4">Mudah (4x4)</option>
            <option value="6x6">Sedang (6x6)</option>
            <option value="8x8">Sulit (8x8)</option>
        </select>
        <button id="startGame">Mulai Game</button>
    </div>
    <div id="game-board"></div>
    <div id="info">
        <div id="timer">Waktu: 00:00</div>
        <div id="steps">Langkah: 0</div>
        <div id="message"></div>
    </div>
    <script>
        const board = document.getElementById('game-board');
        const message = document.getElementById('message');
        const timerElement = document.getElementById('timer');
        const stepsElement = document.getElementById('steps');
        const difficultySelect = document.getElementById('difficulty');
        const startButton = document.getElementById('startGame');

        let colors = [];
        let cards = [];
        let firstCard = null;
        let secondCard = null;
        let lockBoard = false;
        let matchedPairs = 0;
        let steps = 0;
        let timer = null;
        let seconds = 0;
        let minutes = 0;

        // Function to shuffle an array
        function shuffle(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        // Function to start the game
        function startGame() {
            clearInterval(timer);
            timerElement.textContent = 'Waktu: 00:00';
            steps = 0;
            stepsElement.textContent = 'Langkah: 0';
            message.textContent = '';
            matchedPairs = 0;
            cards = [];
            
            const difficulty = difficultySelect.value;
            let gridSize;
            
            switch (difficulty) {
                case '4x4':
                    gridSize = 4;
                    break;
                case '6x6':
                    gridSize = 6;
                    break;
                case '8x8':
                    gridSize = 8;
                    break;
                default:
                    gridSize = 4;
                    break;
            }
            
            colors = Array.from({ length: (gridSize * gridSize) / 2 }, (_, i) => String.fromCodePoint(0x1F34F + i % 5)).flatMap(color => [color, color]);
            setupBoard(gridSize);
            startTimer();
        }

        // Function to setup the game board
        function setupBoard(gridSize) {
            const shuffledColors = shuffle(colors);
            board.innerHTML = '';
            board.style.gridTemplateColumns = `repeat(${gridSize}, 100px)`;
            
            shuffledColors.forEach(color => {
                const card = document.createElement('div');
                card.classList.add('card');
                
                const inner = document.createElement('div');
                inner.classList.add('inner');
                
                const front = document.createElement('div');
                front.classList.add('front');
                inner.appendChild(front);
                
                const back = document.createElement('div');
                back.classList.add('back');
                back.textContent = color;
                inner.appendChild(back);
                
                card.appendChild(inner);
                card.dataset.color = color;
                card.addEventListener('click', flipCard);
                board.appendChild(card);
                cards.push(card);
            });
        }

        // Function to flip a card
        function flipCard() {
            if (lockBoard || this === firstCard) return;
            this.classList.add('flipped');
            
            if (!firstCard) {
                firstCard = this;
            } else {
                secondCard = this;
                checkMatch();
            }
        }

        // Function to check if two cards match
        function checkMatch() {
            steps++;
            stepsElement.textContent = `Langkah: ${steps}`;
            
            if (firstCard.dataset.color === secondCard.dataset.color) {
                matchedPairs++;
                resetBoard();
                if (matchedPairs === colors.length / 2) {
                    clearInterval(timer);
                    message.textContent = `Selamat! Anda menyelesaikan permainan dalam ${minutes} menit ${seconds} detik dan ${steps} langkah.`;
                }
            } else {
                lockBoard = true;
                setTimeout(() => {
                    firstCard.classList.remove('flipped');
                    secondCard.classList.remove('flipped');
                    resetBoard();
                }, 1000);
            }
        }

        // Function to reset the board
        function resetBoard() {
            [firstCard, secondCard, lockBoard] = [null, null, false];
        }

        // Function to start the timer
        function startTimer() {
            seconds = 0;
            minutes = 0;
            timer = setInterval(() => {
                seconds++;
                if (seconds === 60) {
                    seconds = 0;
                    minutes++;
                }
                timerElement.textContent = `Waktu: ${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
            }, 1000);
        }

        // Event listener for the start button
        startButton.addEventListener('click', startGame);

        // Initialize the game
        startGame();
    </script>
</body>
</html>
