<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cat Header Challenge</title>
    <style>
        body {
            margin: 0;
            display: flex;
            justify-content: center; /* Centers horizontally */
            align-items: center;    /* Centers vertically */
            overflow: hidden;
            background-color: #e0e0e0; /* Slightly darker background to contrast the centered game */
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            touch-action: none; /* Prevent scrolling on mobile */
            height: 100vh;
        }

        #game-area {
            /* Responsive Sizing (100vw/vh) remains for small screens */
            width: 100vw;
            height: 100vh;
            
            /* Max Constraints for centering on large screens, preventing infinite stretch */
            max-width: 1000px; 
            max-height: 800px;
            
            position: relative;
            
            /* Added frame aesthetics */
            border: 4px solid #424242; /* Dark border */
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);

            overflow: hidden; /* Keep game elements inside */
            background-color: #f0f8ff; /* Sky for the playing field */
            cursor: crosshair;
            
            /* Visual Floor (20px high) */
            border-bottom: 20px solid;
            border-image: linear-gradient(to right, #4CAF50, #8BC34A) 1;
            
            /* NEW: Shift the game area slightly up and left from the center */
            transform: translate(-5%, -5%); 
        }

        /* UI elements must be relative to the game area */
        #instructions {
            position: absolute;
            top: 20px;
            width: 100%;
            text-align: center;
            color: #546e7a;
            font-size: 1.2rem;
            pointer-events: none;
            user-select: none;
            z-index: 10;
        }

        #score-board {
            position: absolute;
            top: 20px;
            left: 20px;
            font-size: 2rem;
            font-weight: bold;
            color: #333;
            z-index: 20;
            background: rgba(255, 255, 255, 0.8);
            padding: 10px 20px;
            border-radius: 15px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        #high-score-board {
            position: absolute;
            top: 20px;
            right: 20px;
            font-size: 1.5rem;
            font-weight: bold;
            color: #E91E63;
            z-index: 20;
            background: rgba(255, 255, 255, 0.8);
            padding: 10px 20px;
            border-radius: 15px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        .score-gain {
            color: #4CAF50 !important;
            animation: pop 0.3s ease-out;
        }

        .score-loss {
            color: #F44336 !important;
            animation: shake 0.3s ease-out;
        }

        @keyframes pop {
            0% { transform: scale(1); }
            50% { transform: scale(1.3); }
            100% { transform: scale(1); }
        }

        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            25% { transform: translateX(-5px); }
            75% { transform: translateX(5px); }
        }

        /* The Cat Container */
        #cat-container {
            position: absolute;
            width: 100px;
            height: 100px;
            /* Center the cat on its coordinate (for movement) */
            transform: translate(-50%, -50%);
            z-index: 5;
            transition: none; /* Removed transition for crisp game movement */
        }

        /* The SVG Cat */
        #cat-svg {
            width: 100%;
            height: 100%;
            overflow: visible;
        }

        /* Animations */
        .walking #body-group {
            animation: bounce 0.4s infinite alternate ease-in-out;
        }

        .walking #tail {
            animation: tail-wag 0.4s infinite alternate ease-in-out;
        }

        .walking #leg-front-left, .walking #leg-back-right {
            animation: leg-move 0.4s infinite alternate;
        }

        .walking #leg-front-right, .walking #leg-back-left {
            animation: leg-move 0.4s infinite alternate-reverse;
        }

        @keyframes bounce {
            0% { transform: translateY(0); }
            100% { transform: translateY(-3px); }
        }

        @keyframes tail-wag {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(10deg); }
        }

        @keyframes leg-move {
            0% { transform: translateY(0); }
            100% { transform: translateY(-4px); }
        }

        /* Face features blink */
        #eyes {
            animation: blink 4s infinite;
        }

        @keyframes blink {
            0%, 48%, 52%, 100% { transform: scaleY(1); }
            50% { transform: scaleY(0.1); }
        }

        /* Shadow */
        #shadow {
            fill: rgba(0,0,0,0.2);
            transform: scale(1);
            transition: transform 0.2s;
        }
        .walking #shadow {
            transform: scale(0.9);
            opacity: 0.8;
        }

        /* Ball Styles */
        #ball {
            position: absolute;
            width: 40px;
            height: 40px;
            background: radial-gradient(circle at 30% 30%, #ff8a80, #d32f2f);
            border-radius: 50%;
            z-index: 6;
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
            left: 0;
            top: 0;
        }
    </style>
</head>
<body>

    <div id="game-area">
        <div id="score-board">Score: 0</div>
        <div id="high-score-board">Best: 0</div>
        <div id="instructions">Use <b>A</b> (Left) and <b>D</b> (Right) to move! <br> Bounce ball on cat: +1 <br> Drop on ground: -2</div>
        
        <div id="cat-container" style="left: 50%; top: 50%;">
            <svg id="cat-svg" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                <g id="cat-group">
                    <!-- Shadow -->
                    <ellipse id="shadow" cx="50" cy="90" rx="25" ry="5" />

                    <!-- Back Legs -->
                    <path id="leg-back-left" d="M35 80 L35 90" stroke="#333" stroke-width="4" stroke-linecap="round" />
                    <path id="leg-back-right" d="M65 80 L65 90" stroke="#333" stroke-width="4" stroke-linecap="round" />

                    <!-- Tail -->
                    <path id="tail" d="M80 60 Q95 40 85 30" stroke="#FFA726" stroke-width="6" fill="none" stroke-linecap="round" style="transform-origin: 80px 60px;" />

                    <!-- Body Group (bounces when walking) -->
                    <g id="body-group">
                        <!-- Main Body -->
                        <ellipse cx="50" cy="65" rx="30" ry="22" fill="#FFA726" stroke="#E65100" stroke-width="2" />
                        
                        <!-- Front Legs (attached to body) -->
                        <path id="leg-front-left" d="M40 80 L40 90" stroke="#333" stroke-width="4" stroke-linecap="round" />
                        <path id="leg-front-right" d="M60 80 L60 90" stroke="#333" stroke-width="4" stroke-linecap="round" />

                        <!-- Head -->
                        <g id="head">
                            <!-- Ears -->
                            <path d="M35 35 L25 15 L50 30 Z" fill="#FFA726" stroke="#E65100" stroke-width="2" />
                            <path d="M65 35 L75 15 L50 30 Z" fill="#FFA726" stroke="#E65100" stroke-width="2" />
                            
                            <!-- Face Base -->
                            <circle cx="50" cy="40" r="20" fill="#FFB74D" stroke="#E65100" stroke-width="2" />
                            
                            <!-- Eyes and Mouth -->
                            <g id="eyes" style="transform-origin: 50px 40px;">
                                <circle cx="42" cy="36" r="2.5" fill="#333" />
                                <circle cx="58" cy="36" r="2.5" fill="#333" />
                            </g>
                            
                            <!-- Snout -->
                            <ellipse cx="50" cy="45" rx="6" ry="4" fill="#FFF3E0" />
                            <path d="M47 45 Q50 48 53 45" stroke="#333" stroke-width="1.5" fill="none" />
                            <circle cx="50" cy="43" r="1.5" fill="#E91E63" />

                            <!-- Whiskers -->
                            <path d="M30 42 L20 40 M30 45 L20 45 M30 48 L20 50" stroke="#333" stroke-width="1" opacity="0.6" />
                            <path d="M70 42 L80 40 M70 45 L80 45 M70 48 L80 50" stroke="#333" stroke-width="1" opacity="0.6" />
                        </g>
                    </g>
                </g>
            </svg>
        </div>

        <!-- The Ball -->
        <div id="ball"></div>
    </div>

    <script>
        const catContainer = document.getElementById('cat-container');
        const catGroup = document.getElementById('cat-group');
        const catSvg = document.getElementById('cat-svg');
        const gameArea = document.getElementById('game-area');
        const ball = document.getElementById('ball');
        const scoreBoard = document.getElementById('score-board');
        const highScoreBoard = document.getElementById('high-score-board');

        // GAME CONSTANTS
        const FLOOR_MARGIN = 20; // Extra pixels above the bottom border for the ball to bounce
        const CAT_SIZE = 100;
        const BALL_SIZE = 40;
        
        let score = 0;
        let highScore = localStorage.getItem('catGameHighScore') || 0;
        highScoreBoard.innerText = "Best: " + highScore;

        // Physics Variables
        const baseGravity = 0.8; // Faster fall speed
        let gravity = baseGravity; 
        let ballX = 0; 
        let ballY = 0; 
        let ballVx = 0; 
        let ballVy = 0; 

        // Cat State
        let catX = 0;
        let catY = 0; 
        let targetX = null;
        let speed = 15; // Faster cat movement
        let isMoving = false;
        let facingRight = true;
        let floorY = 0;

        /**
         * Resets the ball to a starting position at the top of the screen.
         */
        function resetBall() {
            const gameWidth = gameArea.clientWidth;
            // Spawn ball at a random horizontal position, just above the screen
            ballX = Math.random() * (gameWidth - BALL_SIZE);
            ballY = -BALL_SIZE; 
            ballVx = 0;
            ballVy = 0; // Start with no velocity
        }


        /**
         * Initializes game boundaries and resets positions based on current game area size.
         */
        function initializeGameDimensions() {
            // Get current internal game dimensions
            const gameWidth = gameArea.clientWidth;
            const gameHeight = gameArea.clientHeight;
            
            // Floor Y is the game height MINUS the visual margin MINUS half the cat height (to center the cat)
            floorY = gameHeight - CAT_SIZE / 2 - FLOOR_MARGIN; 
            
            // Reset cat position to the center of the bottom
            // Only update catY, keep catX where it was for smooth resizing
            catY = floorY; 
            
            // If the game is resized, ensure the ball and cat stay within bounds
            catX = Math.min(Math.max(catX, CAT_SIZE / 2), gameWidth - CAT_SIZE / 2);
            ballX = Math.min(Math.max(ballX, 0), gameWidth - BALL_SIZE);
            ballY = Math.min(Math.max(ballY, -BALL_SIZE), gameHeight - BALL_SIZE);
            
            // Re-apply cat position
            catContainer.style.left = catX + 'px';
            catContainer.style.top = catY + 'px';
        }

        function updateScore(change) {
            score += change;
            scoreBoard.innerText = "Score: " + score;
            
            // Visual Feedback
            scoreBoard.className = ""; 
            void scoreBoard.offsetWidth; 
            if (change > 0) {
                scoreBoard.classList.add("score-gain");
            } else {
                scoreBoard.classList.add("score-loss");
            }

            // Update High Score
            if (score > highScore) {
                highScore = score;
                localStorage.setItem('catGameHighScore', highScore);
                highScoreBoard.innerText = "Best: " + highScore;
                highScoreBoard.classList.add("score-gain");
            }

            // Difficulty Progression: Increase gravity as score goes up
            if (score > 0) {
                gravity = baseGravity + (score * 0.02);
                if (gravity > 1.5) gravity = 1.5;
            } else {
                gravity = baseGravity;
            }
        }

        // Keyboard State
        const keys = {
            ArrowLeft: false, ArrowRight: false,
            a: false, d: false
        };

        window.addEventListener('keydown', (e) => {
            const key = e.key.toLowerCase();
            if (keys.hasOwnProperty(key) || keys.hasOwnProperty(e.key)) {
                keys[key] = true;
                targetX = null; // Disable touch target when key is pressed
            }
        });

        window.addEventListener('keyup', (e) => {
            const key = e.key.toLowerCase();
            if (keys.hasOwnProperty(key) || keys.hasOwnProperty(e.key)) {
                keys[key] = false;
            }
        });

        gameArea.addEventListener('pointerdown', (e) => {
            const rect = gameArea.getBoundingClientRect();
            // Target X position relative to the game area
            targetX = e.clientX - rect.left;
        });

        function update() {
            const gameWidth = gameArea.clientWidth;
            const gameHeight = gameArea.clientHeight;
            let dx = 0;
            
            // Lock cat Y position to the floor
            catY = floorY;

            // 1. Handle Keyboard
            if (keys.ArrowLeft || keys.a) dx -= speed;
            if (keys.ArrowRight || keys.d) dx += speed;

            // 2. Handle Click Target (Horizontal movement only)
            if (dx === 0 && targetX !== null) {
                const distX = targetX - catX;
                
                if (Math.abs(distX) > speed) {
                    dx = (distX / Math.abs(distX)) * speed;
                } else {
                    catX = targetX;
                    targetX = null;
                }
            }

            // 3. Apply Cat Movement
            if (dx !== 0) {
                catX += dx;
                isMoving = true;
                const padding = CAT_SIZE / 2;
                // Boundary check: Cat must stay within the game area
                catX = Math.max(padding, Math.min(gameWidth - padding, catX));
                if (dx > 0) facingRight = true;
                if (dx < 0) facingRight = false;
            } else {
                isMoving = false;
            }

            // 4. Ball Physics & Interactions
            ballVy += gravity;
            ballY += ballVy;
            ballX += ballVx;

            // --- Collision with Cat ---
            // Cat's head/body top is about 30px up from its center Y
            const catHeadY = catY - (CAT_SIZE / 2) - 10; 
            const ballCenterX = ballX + BALL_SIZE / 2;
            const ballCenterY = ballY + BALL_SIZE / 2;
            
            const distX = ballCenterX - catX;
            const distY = ballCenterY - catHeadY;
            const distance = Math.sqrt(distX*distX + distY*distY);

            // Collision radius is about 60 for tight head contact.
            if (distance < 60) {
                // Collision!
                if (ballVy > -5) { 
                    // Bounce up 
                    ballVy = -19 - (gravity - 0.8) * 2; 
                    
                    // Deflect horizontally based on hit location - INCREASED COEFFICIENT
                    ballVx = distX * 0.3; // Changed from 0.2 to 0.3
                    
                    // Prevent sticking by forcing ball above cat
                    ballY = catHeadY - BALL_SIZE - 5; 

                    // Score
                    updateScore(1);
                }
            }

            // --- Floor Interaction (Ensures ball stays in play after a miss) ---
            // The ball's bottom edge (Y + BALL_SIZE) hits the floor level
            const floorHitY = gameHeight - BALL_SIZE - FLOOR_MARGIN; 
            if (ballY > floorHitY) {
                // Check if it hit the ground with force (falling) to apply penalty once
                if (ballVy > 2) {
                    updateScore(-2);
                }
                
                // 1. Position the ball exactly on the floor plane
                ballY = floorHitY;
                
                // 2. Use a fixed, moderate upward velocity for a predictable bounce (Was a multiplier)
                ballVy = -13; 
                
                // 3. Set horizontal velocity to zero immediately, making the ball perfectly reachable. (Was 0.1 multiplier)
                ballVx = 0; 
            }
            
            // --- Wall Interaction ---
            if (ballX < 0) {
                ballX = 0;
                ballVx *= -0.8;
            } else if (ballX > gameWidth - BALL_SIZE) {
                ballX = gameWidth - BALL_SIZE;
                ballVx *= -0.8;
            }

            // 5. Update DOM Positions
            ball.style.transform = `translate(${ballX}px, ${ballY}px)`;
            catContainer.style.left = catX + 'px';
            catContainer.style.top = catY + 'px';

            // Flip Cat
            if (facingRight) {
                catGroup.setAttribute('transform', 'scale(1, 1) translate(0, 0)');
            } else {
                // translate(-100, 0) is required to offset the scale(-1) flip so the cat remains in place
                catGroup.setAttribute('transform', 'scale(-1, 1) translate(-100, 0)'); 
            }

            // Walking Animation
            if (isMoving) {
                catSvg.classList.add('walking');
            } else {
                catSvg.classList.remove('walking');
            }

            requestAnimationFrame(update);
        }

        // Initialize dimensions and start the game loop
        window.onload = () => {
            initializeGameDimensions();
            catX = gameArea.clientWidth / 2; // Center the cat initially
            resetBall();
            update();
        };

        // Recalibrate on resize, but don't reset score
        window.addEventListener('resize', initializeGameDimensions);
        
    </script>
</body>
</html>
