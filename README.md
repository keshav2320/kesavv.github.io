<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Holographic Earth AR</title>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/control_utils/control_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>

    <style>
        body { margin: 0; background-color: #000; overflow: hidden; font-family: 'Courier New', Courier, monospace; }
        
        /* Video aur Canvas ko overlap karna */
        .container { position: relative; width: 100vw; height: 100vh; }
        video { position: absolute; width: 100%; height: 100%; object-fit: cover; transform: scaleX(-1); display: none; }
        canvas { position: absolute; width: 100%; height: 100%; object-fit: cover; transform: scaleX(-1); }

        /* Sci-Fi HUD Text Design */
        .hud-text {
            position: absolute;
            color: #00ffcc;
            text-shadow: 0 0 5px #00ffcc;
            font-weight: bold;
            z-index: 10;
            pointer-events: none;
        }
        #sys-scan { top: 20px; left: 20px; font-size: 18px; animation: blink 1s infinite; }
        #bio-link { top: 50px; left: 20px; font-size: 14px; }
        
        @keyframes blink { 50% { opacity: 0.5; } }
    </style>
</head>
<body>

    <div class="container">
        <div id="sys-scan" class="hud-text">| SYS_SCANNING...</div>
        <div id="bio-link" class="hud-text">| BIO_LINK: SEARCHING</div>

        <video id="input_video"></video>
        <canvas id="output_canvas"></canvas>
    </div>

    <script>
        const videoElement = document.getElementById('input_video');
        const canvasElement = document.getElementById('output_canvas');
        const canvasCtx = canvasElement.getContext('2d');
        const bioLinkText = document.getElementById('bio-link');

        // Earth Image Load karna (Hologram ke liye)
        const earthImg = new Image();
        earthImg.src = 'https://upload.wikimedia.org/wikipedia/commons/thumb/2/22/Earth_Western_Hemisphere_transparent_background.png/1200px-Earth_Western_Hemisphere_transparent_background.png';
        
        // Rotation variable
        let angle = 0;

        function onResults(results) {
            // Canvas ko clear aur size set karna
            canvasElement.width = window.innerWidth;
            canvasElement.height = window.innerHeight;
            
            canvasCtx.save();
            canvasCtx.clearRect(0, 0, canvasElement.width, canvasElement.height);
            
            // 1. Camera feed draw karna
            canvasCtx.drawImage(results.image, 0, 0, canvasElement.width, canvasElement.height);

            if (results.multiHandLandmarks && results.multiHandLandmarks.length > 0) {
                // Haath mil gaya!
                bioLinkText.innerText = "| BIO_LINK: STABLE";
                bioLinkText.style.color = "#00ffcc";

                for (const landmarks of results.multiHandLandmarks) {
                    // 2. Green Tech Lines draw karna (Skeleton)
                    drawConnectors(canvasCtx, landmarks, HAND_CONNECTIONS,
                                   {color: '#00ffcc', lineWidth: 2});
                    drawLandmarks(canvasCtx, landmarks, 
                                   {color: '#00ffcc', lineWidth: 1, radius: 3});

                    // 3. Hatheli (Palm) ka center nikaalna (Wrist aur Middle Finger ke beech)
                    const wrist = landmarks[0];
                    const middleFinger = landmarks[9];
                    
                    // Coordinates ko pixel mein convert karna
                    const palmX = (wrist.x + middleFinger.x) / 2 * canvasElement.width;
                    const palmY = (wrist.y + middleFinger.y) / 2 * canvasElement.height;

                    // 4. Earth ko Palm ke upar draw karna
                    const size = 150; // Earth ka size
                    
                    canvasCtx.translate(palmX, palmY - 100); // Haath se thoda upar float karega
                    canvasCtx.rotate(angle); // Earth ko ghumana
                    canvasCtx.drawImage(earthImg, -size/2, -size/2, size, size);
                    canvasCtx.rotate(-angle); // Reset rotation
                    canvasCtx.translate(-palmX, -(palmY - 100)); // Reset position

                    // Agli frame ke liye angle badhana
                    angle += 0.05;
                }
            } else {
                bioLinkText.innerText = "| BIO_LINK: SEARCHING...";
                bioLinkText.style.color = "red";
            }
            canvasCtx.restore();
        }

        // MediaPipe Hands Setup
        const hands = new Hands({locateFile: (file) => {
            return `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`;
        }});
        
        hands.setOptions({
            maxNumHands: 1,
            modelComplexity: 1,
            minDetectionConfidence: 0.5,
            minTrackingConfidence: 0.5
        });
        
        hands.onResults(onResults);

        // Camera start karna
        const camera = new Camera(videoElement, {
            onFrame: async () => {
                await hands.send({image: videoElement});
            },
            width: 1280,
            height: 720
        });
        camera.start();
    </script>
</body>
</html>
