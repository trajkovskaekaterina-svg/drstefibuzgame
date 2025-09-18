[index.html](https://github.com/user-attachments/files/22407658/index.html)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dr. Stefi's Exam Run</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/tone@14.7.58/build/Tone.js"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8;
            color: #1e293b;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 1rem;
            overflow: hidden;
        }

        @keyframes flashing-lights {
            0% { background-color: #f0f4f8; }
            25% { background-color: #ff9a8b; }
            50% { background-color: #90ee90; }
            75% { background-color: #add8e6; }
            100% { background-color: #f0f4f8; }
        }

        .flashing-background {
            animation: flashing-lights 1s infinite alternate;
        }

        .game-container {
            position: relative;
            width: 100%;
            max-width: 900px;
            aspect-ratio: 16 / 9;
            border-radius: 1.5rem;
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
            overflow: hidden;
            background: linear-gradient(to bottom, #87CEEB, #A0CBE7); /* Sky gradient */
        }
        
        #game-canvas {
            display: block;
            width: 100%;
            height: 100%;
            background-color: #f0f4f8;
        }

        .overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            text-align: center;
            background-color: rgba(255, 255, 255, 0.85);
            backdrop-filter: blur(8px);
            transition: opacity 0.5s ease-in-out;
            padding: 2rem;
            border-radius: 1.5rem;
            z-index: 10;
        }
        
        .overlay.hidden {
            opacity: 0;
            pointer-events: none;
        }

        .button {
            padding: 0.75rem 2rem;
            font-weight: bold;
            border-radius: 9999px;
            transition: background-color 0.3s, transform 0.2s, box-shadow 0.3s;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
        }

        .button:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }

        .exams-counter {
            position: absolute;
            top: 1rem;
            left: 1rem;
            font-size: 1.5rem;
            font-weight: bold;
            color: #1e293b;
            background-color: rgba(255, 255, 255, 0.7);
            padding: 0.5rem 1rem;
            border-radius: 1rem;
            z-index: 5;
        }
        #music-control {
            margin-top: 1rem;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 0.5rem;
        }
    </style>
</head>
<body class="bg-gradient-to-br from-purple-200 to-indigo-300">

<div class="game-container">
    <canvas id="game-canvas"></canvas>

    <!-- Start/End Screen Overlay -->
    <div id="game-overlay" class="overlay">
        <h1 id="main-title" class="text-4xl md:text-6xl lg:text-7xl font-extrabold text-blue-800 leading-tight">
            Dr. Stefi's Exam Run
        </h1>
        <p id="main-text" class="mt-4 text-lg md:text-xl lg:text-2xl font-medium text-gray-700 max-w-lg">
            help Dr. Stefi pass all 61 of her exams by collecting prizes and jumping over obstacles ! ( soberi 61 emoji - AKO MISLIS DEKA SE MNOGU ZAMISLI KAKO BILO NA STEFI DA POLOZI 61 ISPIT )
        </p>
        <button id="start-button" class="mt-8 button bg-blue-600 text-white hover:bg-blue-700 focus:outline-none focus:ring-4 focus:ring-blue-300">
            Start Game
        </button>
        <button id="toggle-music-button" class="mt-4 button bg-purple-600 text-white hover:bg-purple-700 focus:outline-none focus:ring-4 focus:ring-purple-300">
            Play Music
        </button>
    </div>

    <!-- Exam Counter -->
    <div id="exams-counter" class="exams-counter hidden">
        Exams: <span id="exams-passed">0</span> / 61
    </div>
</div>

<script>
    // Game variables and constants
    const canvas = document.getElementById('game-canvas');
    const ctx = canvas.getContext('2d');
    const overlay = document.getElementById('game-overlay');
    const startButton = document.getElementById('start-button');
    const toggleMusicButton = document.getElementById('toggle-music-button');
    const examsCounter = document.getElementById('exams-counter');
    const examsPassedSpan = document.getElementById('exams-passed');
    const body = document.body;

    let gameRunning = false;
    let examsPassed = 0;
    const MAX_EXAMS = 61;
    let lastTime = 0;
    
    // Player settings
    const player = {
        x: 50,
        y: 0,
        width: 100,
        height: 100,
        velocityY: 0,
        gravity: 0.8,
        isJumping: false,
        image: new Image()
    };
    
    player.image.onload = () => {
        setupCanvas();
        overlay.classList.remove('hidden'); // Show overlay after image loads
    };
    player.image.onerror = () => {
        console.error("Failed to load player image. Check the URL.");
        // Fallback to a simple colored rectangle if image fails to load
        player.image = null;
        setupCanvas();
        overlay.classList.remove('hidden');
    };
    player.image.src = 'https://i.postimg.cc/52vnSCZT/Subject-4.png';
    
    // Ground and background settings
    const ground = {
        y: 0,
        height: 20,
        color: '#16A34A',
    };
    let backgroundX = 0;
    let cloudsX = 0;
    let hillsX = 0;
    const backgroundSpeed = 0.5;
    const cloudSpeed = 0.1;
    const hillsSpeed = 0.25;

    // Obstacle and collectible settings
    let gameItems = [];
    let itemSpeed = 5;
    let itemSpawnInterval = 1000;
    let lastItemSpawn = 0;
    const collectibles = ['10ka', '💊', '💉', '🩹', '🌡️', '🩺'];
    const obstacles = ['🛑', '⚠️', '❌']; // New obstacle emojis

    // Personal messages at scores
    const messages = {
        10: "POLOZI 10 ISPITI VREME ZA NA KAFE VO PUBLIC.",
        20: "POLOZI 20 ISPITI VREME E ZA NA PIJACKA.",
        30: "NA POLA PAT SI ! BRAVO",
        40: "MNOGU BILO MALCE OSTANALO .",
        50: "AJDE STEFIIIIIII ! SITNO SE BROI TE CEKAME SO DIPLOMATA VO RACE.",
        61: "DR. STEFANI BUZAROVSKA LADIES AND GENTS !"
    };
    let lastMessageShown = 0;

    // Confetti particles
    let confetti = [];
    let showConfetti = false;
    const confettiColors = ['#f44336', '#e91e63', '#9c27b0', '#673ab7', '#3f51b5', '#2196f3', '#03a9f4', '#00bcd4', '#009688', '#4caf50', '#8bc34a', '#cddc39', '#ffeb3b', '#ffc107', '#ff9800', '#ff5722'];

    class Confetti {
        constructor(x, y, size, speedX, speedY, color) {
            this.x = x;
            this.y = y;
            this.size = size;
            this.speedX = speedX;
            this.speedY = speedY;
            this.color = color;
            this.rotation = Math.random() * 360;
        }

        update() {
            this.speedY += 0.1; // Gravity
            this.x += this.speedX;
            this.y += this.speedY;
            this.rotation += this.speedX;
        }

        draw(ctx) {
            ctx.fillStyle = this.color;
            ctx.save();
            ctx.translate(this.x, this.y);
            ctx.rotate(this.rotation * Math.PI / 180);
            ctx.fillRect(-this.size / 2, -this.size / 2, this.size, this.size);
            ctx.restore();
        }
    }

    function createConfetti() {
        for (let i = 0; i < 20; i++) {
            const size = Math.random() * 8 + 4;
            const speedX = (Math.random() - 0.5) * 5;
            const speedY = -(Math.random() * 5 + 5);
            const color = confettiColors[Math.floor(Math.random() * confettiColors.length)];
            confetti.push(new Confetti(Math.random() * canvas.width, canvas.height, size, speedX, speedY, color));
        }
    }

    function drawConfetti() {
        confetti.forEach((c, index) => {
            c.update();
            c.draw(ctx);
            if (c.y > canvas.height) {
                confetti.splice(index, 1);
            }
        });

        // Continuously create new confetti particles if visible
        if (showConfetti && confetti.length < 50) {
            createConfetti();
        }
    }

    // Music setup
    const synth = new Tone.PolySynth(Tone.Synth).toDestination();
    const chords = [
        ["C4", "E4", "G4"],
        ["F4", "A4", "C5"],
        ["G4", "B4", "D5"],
        ["C4", "E4", "G4"]
    ];
    let musicSequence;
    let musicStarted = false;

    function startMusic() {
        if (!musicStarted) {
            musicStarted = true;
            musicSequence = new Tone.Sequence((time, chord) => {
                synth.triggerAttackRelease(chord, "4n", time);
            }, chords, "4n").start(0);
            Tone.Transport.start();
        }
    }

    function stopMusic() {
        if (musicStarted) {
            musicSequence.stop();
            musicSequence.dispose();
            Tone.Transport.stop();
            musicStarted = false;
        }
    }

    // Adjust canvas and ground position on resize
    function setupCanvas() {
        canvas.width = canvas.parentElement.clientWidth;
        canvas.height = canvas.parentElement.clientHeight;
        ground.y = canvas.height - ground.height;
        player.y = ground.y - player.height;
    }
    
    // Event listener for jumping
    function handleJump() {
        if (!player.isJumping) {
            player.velocityY = -15; // Jump strength
            player.isJumping = true;
        }
    }

    // Function to draw clouds
    function drawClouds() {
        ctx.fillStyle = 'rgba(255, 255, 255, 0.8)';
        ctx.beginPath();
        ctx.arc(canvas.width * 0.2 + cloudsX, canvas.height * 0.2, 30, 0, Math.PI * 2);
        ctx.arc(canvas.width * 0.2 + 25 + cloudsX, canvas.height * 0.2, 25, 0, Math.PI * 2);
        ctx.arc(canvas.width * 0.2 - 20 + cloudsX, canvas.height * 0.2 + 10, 20, 0, Math.PI * 2);
        ctx.arc(canvas.width * 0.2 + 10 + cloudsX, canvas.height * 0.2 + 15, 25, 0, Math.PI * 2);
        ctx.fill();

        ctx.beginPath();
        ctx.arc(canvas.width * 0.7 + cloudsX, canvas.height * 0.3, 40, 0, Math.PI * 2);
        ctx.arc(canvas.width * 0.7 + 30 + cloudsX, canvas.height * 0.3, 35, 0, Math.PI * 2);
        ctx.arc(canvas.width * 0.7 - 25 + cloudsX, canvas.height * 0.3 + 20, 30, 0, Math.PI * 2);
        ctx.arc(canvas.width * 0.7 + 15 + cloudsX, canvas.height * 0.3 + 25, 40, 0, Math.PI * 2);
        ctx.fill();

        cloudsX -= cloudSpeed;
        if (cloudsX < -canvas.width) {
            cloudsX = 0;
        }
    }

    // Function to draw hills
    function drawHills() {
        ctx.fillStyle = '#6B8E23';
        ctx.beginPath();
        ctx.moveTo(0 + hillsX, ground.y);
        ctx.lineTo(canvas.width * 0.25 + hillsX, ground.y);
        ctx.lineTo(canvas.width * 0.35 + hillsX, ground.y - 50);
        ctx.lineTo(canvas.width * 0.45 + hillsX, ground.y);
        ctx.lineTo(canvas.width * 0.7 + hillsX, ground.y);
        ctx.lineTo(canvas.width * 0.8 + hillsX, ground.y - 70);
        ctx.lineTo(canvas.width + hillsX, ground.y);
        ctx.lineTo(0 + hillsX, ground.y);
        ctx.fill();

        hillsX -= hillsSpeed;
        if (hillsX < -canvas.width) {
            hillsX = 0;
        }
    }

    // Function to draw the simple hospital and MEDF buildings
    function drawBuildings() {
        // Draw main hospital building
        ctx.fillStyle = '#6B7280'; // Main building color
        ctx.fillRect(canvas.width - 200 + backgroundX, ground.y - 200, 150, 200);
        
        // Draw windows
        ctx.fillStyle = '#C7D2FE'; // Window color
        ctx.fillRect(canvas.width - 180 + backgroundX, ground.y - 180, 20, 30);
        ctx.fillRect(canvas.width - 140 + backgroundX, ground.y - 180, 20, 30);
        ctx.fillRect(canvas.width - 180 + backgroundX, ground.y - 140, 20, 30);
        ctx.fillRect(canvas.width - 140 + backgroundX, ground.y - 140, 20, 30);
        
        // Draw cross
        ctx.fillStyle = 'red';
        ctx.fillRect(canvas.width - 125 + backgroundX, ground.y - 200, 10, 15);
        ctx.fillRect(canvas.width - 130 + backgroundX, ground.y - 195, 20, 5);
        
        // Draw the MEDF building
        const medfBuildingX = canvas.width - 400 + backgroundX;
        const medfBuildingY = ground.y - 250;
        const medfBuildingWidth = 200;
        const medfBuildingHeight = 250;

        ctx.fillStyle = '#A0CBE7';
        ctx.fillRect(medfBuildingX, medfBuildingY, medfBuildingWidth, medfBuildingHeight);

        // Draw windows on MEDF building
        ctx.fillStyle = '#C7D2FE';
        for (let i = 0; i < 4; i++) {
            for (let j = 0; j < 3; j++) {
                ctx.fillRect(medfBuildingX + 20 + (i * 45), medfBuildingY + 20 + (j * 50), 25, 40);
            }
        }

        // Add the "MEDF" text
        ctx.fillStyle = '#1e293b';
        ctx.font = 'bold 30px Arial';
        ctx.textAlign = 'center';
        ctx.fillText('MEDF', medfBuildingX + medfBuildingWidth / 2, medfBuildingY + medfBuildingHeight - 30);
        ctx.textAlign = 'left'; // Reset text alignment

        backgroundX -= backgroundSpeed;
        if (backgroundX < -canvas.width) {
            backgroundX = 0;
        }
    }

    // Function to display a temporary message on the canvas
    function displayMessage(text) {
        const messageDuration = 2000;
        const messageElement = document.createElement('div');
        messageElement.textContent = text;
        messageElement.style.position = 'absolute';
        messageElement.style.top = '50%';
        messageElement.style.left = '50%';
        messageElement.style.transform = 'translate(-50%, -50%)';
        messageElement.style.color = 'white';
        messageElement.style.fontSize = '3rem';
        messageElement.style.fontWeight = 'bold';
        messageElement.style.textShadow = '2px 2px 4px rgba(0,0,0,0.7)';
        messageElement.style.zIndex = '20';
        messageElement.style.opacity = '1';
        messageElement.style.transition = 'opacity 0.5s';
        
        document.querySelector('.game-container').appendChild(messageElement);

        setTimeout(() => {
            messageElement.style.opacity = '0';
            setTimeout(() => {
                messageElement.remove();
            }, 500);
        }, messageDuration);
    }


    // Main game loop
    function gameLoop(timestamp) {
        if (!gameRunning) return;

        const deltaTime = timestamp - lastTime;
        lastTime = timestamp;

        // Clear the canvas
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        // Draw the background layers
        drawHills();
        drawClouds();
        drawBuildings();

        // Update and draw ground
        ctx.fillStyle = ground.color;
        ctx.fillRect(0, ground.y, canvas.width, ground.height);

        // Update player position
        player.velocityY += player.gravity;
        player.y += player.velocityY;

        // Prevent player from falling through the ground
        if (player.y > ground.y - player.height) {
            player.y = ground.y - player.height;
            player.velocityY = 0;
            player.isJumping = false;
        }

        // Draw player image
        if (player.image) {
            ctx.drawImage(player.image, player.x, player.y, player.width, player.height);
        } else {
            // Fallback if image fails to load
            ctx.fillStyle = 'blue';
            ctx.fillRect(player.x, player.y, player.width, player.height);
        }

        // Spawn new items (collectibles or obstacles)
        if (examsPassed < MAX_EXAMS && timestamp - lastItemSpawn > itemSpawnInterval) {
            const isCollectible = Math.random() < 0.8; // 80% chance to spawn a collectible
            const itemArray = isCollectible ? collectibles : obstacles;
            const itemValue = itemArray[Math.floor(Math.random() * itemArray.length)];
            
            const itemHeight = 30; // Uniform height for drawing
            const itemWidth = 30; // Uniform width
            
            let itemY;
            if (isCollectible) {
                // Collectibles appear at varying heights
                itemY = ground.y - Math.random() * 120 - 50;
            } else {
                // Obstacles appear on the ground
                itemY = ground.y - itemHeight;
            }
            
            gameItems.push({
                x: canvas.width,
                y: itemY,
                width: itemWidth,
                height: itemHeight,
                value: itemValue,
                isCollectible: isCollectible
            });
            lastItemSpawn = timestamp;
        }

        // Update and draw game items
        gameItems.forEach(item => {
            item.x -= itemSpeed;
            ctx.font = `${item.height}px sans-serif`;
            // Use item.y directly to place the text
            ctx.fillText(item.value, item.x, item.y + item.height - 10);

            // Collision detection
            if (
                player.x < item.x + item.width &&
                player.x + player.width > item.x &&
                player.y + player.height > item.y &&
                player.y < item.y + item.height
            ) {
                if (item.isCollectible) {
                    examsPassed++;
                    examsPassedSpan.innerText = examsPassed;
                    // Check for messages
                    if (messages[examsPassed] && examsPassed !== lastMessageShown) {
                        displayMessage(messages[examsPassed]);
                        lastMessageShown = examsPassed;
                    }
                    if (examsPassed >= MAX_EXAMS) {
                        endGame(true);
                    }
                }
                // Remove the item after collision, regardless of whether it's an obstacle or collectible
                item.x = -100;
            }
        });

        // Remove off-screen items
        gameItems = gameItems.filter(item => item.x + item.width > 0);

        // Draw confetti if it's the end screen
        if (showConfetti) {
            drawConfetti();
        }

        requestAnimationFrame(gameLoop);
    }
    
    // Start game function
    function startGame() {
        gameRunning = true;
        examsPassed = 0;
        gameItems = [];
        player.y = ground.y - player.height;
        player.velocityY = 0;
        player.isJumping = false;
        lastTime = performance.now();
        backgroundX = 0;
        cloudsX = 0;
        hillsX = 0;
        lastMessageShown = 0;
        
        // Hide overlay and show counter
        overlay.classList.add('hidden');
        examsCounter.classList.remove('hidden');
        body.classList.remove('flashing-background');
        stopMusic();
        showConfetti = false;
        confetti = [];

        requestAnimationFrame(gameLoop);
    }

    // End game function
    function endGame(win = false) {
        gameRunning = false;
        
        // Show overlay and reset styles
        overlay.classList.remove('hidden');
        examsCounter.classList.add('hidden');

        const title = document.getElementById('main-title');
        const text = document.getElementById('main-text');
        const startButton = document.getElementById('start-button');
        
        if (win) {
            startMusic();
            body.classList.add('flashing-background');
            title.innerText = "🎉 DR STEFANI BUZAROVSKA DAMI I GOSPODA ! 🎉";
            title.classList.add('animate-pulse');
            text.innerText = "SEKOJ BI POSAKAL DOKTOR KAKO NEA !";
            text.classList.add('text-white', 'text-shadow-md', 'font-bold');
            startButton.innerText = "Play Again";
            startButton.onclick = () => {
                window.location.reload();
            };
            
            // Add music volume control
            const musicControl = document.createElement('div');
            musicControl.id = "music-control";
            musicControl.innerHTML = `
                <label for="volume-slider" class="text-white font-bold">Music Volume</label>
                <input type="range" id="volume-slider" min="0" max="1" step="0.01" value="0.5">
            `;
            overlay.appendChild(musicControl);

            const volumeSlider = document.getElementById('volume-slider');
            volumeSlider.addEventListener('input', (e) => {
                Tone.Destination.volume.value = Tone.gainToDb(e.target.value);
            });
            
            showConfetti = true;
            createConfetti();

        } else {
            title.innerText = "Game Over!";
            text.innerText = `You passed ${examsPassed} out of ${MAX_EXAMS} exams. Keep trying!`;
            startButton.innerText = "Try Again";
            startButton.onclick = () => {
                startGame();
            };
        }
    }

    // Initial setup on window load
    window.onload = function() {
        setupCanvas();
        // Event listeners for jump
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space' && gameRunning) {
                handleJump();
            }
        });
        canvas.addEventListener('mousedown', handleJump);
        canvas.addEventListener('touchstart', handleJump);

        // Start button event listener
        startButton.addEventListener('click', () => {
             // Forcing audio context start on first user interaction
            if (Tone.context.state !== 'running') {
                Tone.context.resume();
            }
            startGame();
        });

        // Toggle music button listener
        let isMusicPlaying = false;
        toggleMusicButton.addEventListener('click', () => {
            if (Tone.context.state !== 'running') {
                Tone.context.resume();
            }
            if (!isMusicPlaying) {
                startMusic();
                toggleMusicButton.innerText = "Stop Music";
            } else {
                stopMusic();
                toggleMusicButton.innerText = "Play Music";
            }
            isMusicPlaying = !isMusicPlaying;
        });

        window.addEventListener('resize', setupCanvas);
    };
</script>

</body>
</html>
S
