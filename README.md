<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Motion Extraction</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        /* General Body & Layout */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #111827;
            color: #e5e7eb;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            padding: 1rem;
            margin: 0;
        }

        .container {
            width: 100%;
            max-width: 56rem; /* Equivalent to max-w-4xl */
            margin-left: auto;
            margin-right: auto;
            text-align: center;
        }

        .hidden {
            display: none;
        }

        /* Hide the default file input button */
        input[type="file"] {
            display: none;
        }

        /* Header */
        header {
            margin-bottom: 2rem;
        }

        h1 {
            font-size: 2.25rem;
            font-weight: 700;
            letter-spacing: -0.025em;
            color: #ffffff;
        }

        header p {
            margin-top: 1rem;
            font-size: 1.125rem;
            color: #9ca3af;
        }
        
        /* Upload Area */
        .custom-file-upload {
            border: 2px dashed #4a5568;
            display: inline-flex;
            align-items: center;
            justify-content: center;
            padding: 2rem 4rem;
            cursor: pointer;
            transition: all 0.3s ease;
            border-radius: 0.5rem;
            color: #9ca3af;
        }

        .custom-file-upload:hover {
            border-color: #a0aec0;
            background-color: #1f2937;
            color: #ffffff;
        }

        .custom-file-upload svg {
            width: 2.5rem;
            height: 2.5rem;
            margin-right: 1rem;
        }

        /* Player & Controls */
        #player-container .canvas-wrapper {
            position: relative;
            background-color: #000;
            border-radius: 0.5rem;
            box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1), 0 4px 6px -2px rgba(0,0,0,0.05);
            overflow: hidden;
        }

        #video-canvas {
            width: 100%;
            height: auto;
            display: block;
        }
        
        .controls-panel {
            margin-top: 1.5rem;
            padding: 1.5rem;
            background-color: #1f2937;
            border-radius: 0.5rem;
        }

        .controls-row-1 {
            display: flex;
            align-items: center;
            justify-content: space-between;
            gap: 2rem;
        }

        .button-group {
            display: flex;
            flex-wrap: wrap;
            align-items: center;
            gap: 0.5rem;
        }

        .slider-group-1 {
            flex-grow: 1;
        }

        .controls-row-2 {
            margin-top: 1rem;
            display: grid;
            grid-template-columns: 1fr;
            gap: 1rem;
        }

        label {
            display: block;
            font-size: 0.875rem;
            font-weight: 500;
            color: #d1d5db;
            margin-bottom: 0.5rem;
            text-align: left;
        }

        label span {
            font-weight: 700;
            color: #ffffff;
        }
        
        /* Buttons */
        .btn {
            padding: 0.5rem 1rem;
            color: #ffffff;
            font-weight: 600;
            border-radius: 0.5rem;
            transition: background-color 0.3s ease;
            white-space: nowrap;
            border: none;
            cursor: pointer;
        }

        .btn-indigo { background-color: #4f46e5; }
        .btn-indigo:hover { background-color: #6366f1; }
        .btn-gray { background-color: #4b5563; }
        .btn-gray:hover { background-color: #6b7280; }
        .btn-green { background-color: #16a34a; }
        .btn-green:hover { background-color: #22c55e; }
        .btn-red { background-color: #dc2626; }
        .btn-red:hover { background-color: #ef4444; }


        /* Note & Link */
        .note {
            font-size: 0.75rem;
            color: #6b7280;
            margin-top: 1rem;
            text-align: left;
        }
        .inspiration-link {
            display: block;
            text-align: left;
            margin-top: 0.5rem;
            font-size: 0.75rem;
            color: #6b7280;
            transition: color 0.3s ease;
        }
        .inspiration-link:hover {
            color: #d1d5db;
        }
        
        /* Custom slider styles */
        input[type=range] {
            -webkit-appearance: none;
            width: 100%;
            background: transparent;
        }
        input[type=range]:focus {
            outline: none;
        }
        input[type=range]::-webkit-slider-runnable-track {
            width: 100%;
            height: 8px;
            cursor: pointer;
            background: #4a5568;
            border-radius: 5px;
        }
        input[type=range]::-webkit-slider-thumb {
            -webkit-appearance: none;
            height: 20px;
            width: 20px;
            border-radius: 50%;
            background: #cbd5e0;
            cursor: pointer;
            margin-top: -6px;
        }
        input[type=range]::-moz-range-track {
            width: 100%;
            height: 8px;
            cursor: pointer;
            background: #4a5568;
            border-radius: 5px;
        }
        input[type=range]::-moz-range-thumb {
            height: 20px;
            width: 20px;
            border-radius: 50%;
            background: #cbd5e0;
            cursor: pointer;
        }
        input[type=range]:disabled::-webkit-slider-thumb {
            background: #718096;
        }
        input[type=range]:disabled::-moz-range-thumb {
            background: #718096;
        }

        /* Responsive */
        @media (min-width: 640px) {
            .btn {
                 padding: 0.5rem 1.5rem;
            }
             .button-group {
                gap: 1rem;
            }
        }
        @media (min-width: 768px) {
            .controls-row-2 {
                grid-template-columns: 1fr 1fr;
            }
             h1 {
                font-size: 3rem;
            }
        }
    </style>

    <!-- Shaders for WebGL -->
    <script id="vertex-shader" type="x-shader/x-vertex">
        attribute vec2 a_position;
        varying vec2 v_texCoord;
        void main() {
            // Convert position to texture coordinates and flip the Y-axis
            vec2 texCoord = a_position * 0.5 + 0.5;
            v_texCoord = vec2(texCoord.x, 1.0 - texCoord.y);
            gl_Position = vec4(a_position, 0.0, 1.0);
        }
    </script>

    <script id="fragment-shader" type="x-shader/x-fragment">
        precision mediump float;
        uniform sampler2D u_video1;
        uniform sampler2D u_video2;
        uniform vec2 u_resolution;
        uniform bool u_edges;
        uniform bool u_glow;
        uniform float u_glowAmount;

        varying vec2 v_texCoord;

        // Calculates the absolute difference between two frames.
        vec4 getDifference(vec2 texCoord) {
            vec4 color1 = texture2D(u_video1, texCoord);
            vec4 color2 = texture2D(u_video2, texCoord);
            vec3 difference = abs(color1.rgb - color2.rgb);
            return vec4(difference, 1.0);
        }

        // Calculates edges based on the motion difference.
        vec4 getEdges(vec2 texCoord) {
            float pixelX = 1.0 / u_resolution.x;
            float pixelY = 1.0 / u_resolution.y;
            
            mat3 sobelX = mat3(-1.0, 0.0, 1.0, -2.0, 0.0, 2.0, -1.0, 0.0, 1.0);
            mat3 sobelY = mat3(-1.0, -2.0, -1.0, 0.0, 0.0, 0.0, 1.0, 2.0, 1.0);
            
            float gradX = 0.0;
            float gradY = 0.0;

            for (int i = -1; i <= 1; i++) {
                for (int j = -1; j <= 1; j++) {
                    vec2 offset = texCoord + vec2(float(i) * pixelX, float(j) * pixelY);
                    // Base edge detection on the standard difference now
                    vec4 sample = getDifference(offset);
                    float gray = dot(sample.rgb, vec3(0.299, 0.587, 0.114));
                    gradX += gray * sobelX[i+1][j+1];
                    gradY += gray * sobelY[i+1][j+1];
                }
            }
            
            float magnitude = sqrt(gradX * gradX + gradY * gradY);
            return vec4(vec3(magnitude), 1.0);
        }

        void main() {
            // Call the corrected getDifference function when edges are off
            vec4 motionColor = u_edges ? getEdges(v_texCoord) : getDifference(v_texCoord);
            
            if (u_glow && u_glowAmount > 0.0) {
                vec4 totalGlow = vec4(0.0);
                float blurSizeX = u_glowAmount / u_resolution.x;
                float blurSizeY = u_glowAmount / u_resolution.y;

                // A simplified blur for performance
                for (int x = -4; x <= 4; x++) {
                    for (int y = -4; y <= 4; y++) {
                        vec2 offset = v_texCoord + vec2(float(x) * blurSizeX, float(y) * blurSizeY);
                        // Calculate glow based on the corrected functions
                        vec4 sample = u_edges ? getEdges(offset) : getDifference(offset);
                        totalGlow += sample;
                    }
                }
                
                gl_FragColor = motionColor + totalGlow / 81.0; // Additive glow
            } else {
                gl_FragColor = motionColor;
            }
        }
    </script>
</head>
<body>

    <div class="container">
        <header>
            <h1>Motion Extraction</h1>
            <p>Upload an MP4, apply filters & offset amount to extract the motion in any video.</p>
        </header>

        <!-- Video Upload Section -->
        <div id="upload-container">
            <label for="video-upload" class="custom-file-upload">
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-4-4V6a4 4 0 014-4h10a4 4 0 014 4v6a4 4 0 01-4 4H7z" />
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                </svg>
                <span>Choose an MP4 file</span>
            </label>
            <input id="video-upload" type="file" accept="video/mp4" />
        </div>

        <!-- Video Player Section (Initially Hidden) -->
        <div id="player-container" class="hidden">
            <div class="canvas-wrapper">
                <canvas id="video-canvas"></canvas>
            </div>
            
            <div class="controls-panel">
                <div class="controls-row-1">
                     <div class="button-group">
                        <button id="play-pause" class="btn btn-indigo">Play</button>
                        <button id="edges-btn" class="btn btn-gray">Edges</button>
                        <button id="glow-btn" class="btn btn-gray">Glow</button>
                        <button id="upload-new-btn" class="btn btn-gray">New Video</button>
                        <button id="download-btn" class="btn btn-green">Download</button>
                    </div>
                    <div class="slider-group-1">
                        <label for="offset-slider">Frame Offset: <span id="offset-value">50</span></label>
                        <input id="offset-slider" type="range" min="1" max="150" value="50">
                    </div>
                </div>
                 <div class="controls-row-2">
                    <div>
                        <label for="speed-slider">Playback Speed: <span id="speed-value">1.00</span>x</label>
                        <input id="speed-slider" type="range" min="0.25" max="2" value="1" step="0.25">
                    </div>
                     <div>
                        <label for="glow-slider">Glow Amount: <span id="glow-value">5</span></label>
                        <input id="glow-slider" type="range" min="0" max="20" value="5" step="1" disabled>
                    </div>
                </div>
                 <p class="note">Note: Offset calculation assumes a video frame rate of 30 FPS. Results may vary for videos with different frame rates. Downloads are in WebM format.</p>
                 <a href="https://www.youtube.com/watch?v=NSS6yAMZF78" target="_blank" rel="noopener noreferrer" class="inspiration-link">Inspired by Posy</a>
                 <a href="https://x.com/Cruel_Coppinger" target="_blank" rel="noopener noreferrer" class="inspiration-link">Vibecoded by @Cruel_Coppinger</a>
            </div>
        </div>
    </div>

    <!-- Hidden video elements for processing -->
    <video id="video1" muted playsinline loop hidden></video>
    <video id="video2" muted playsinline loop hidden></video>
    
    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- Global Variables ---
        const uploadInput = document.getElementById('video-upload');
        const uploadContainer = document.getElementById('upload-container');
        const playerContainer = document.getElementById('player-container');
        
        const video1 = document.getElementById('video1');
        const video2 = document.getElementById('video2');
        const canvas = document.getElementById('video-canvas');
        
        const playPauseBtn = document.getElementById('play-pause');
        const offsetSlider = document.getElementById('offset-slider');
        const offsetValue = document.getElementById('offset-value');
        const uploadNewBtn = document.getElementById('upload-new-btn');
        const speedSlider = document.getElementById('speed-slider');
        const speedValue = document.getElementById('speed-value');
        const glowBtn = document.getElementById('glow-btn');
        const glowSlider = document.getElementById('glow-slider');
        const glowValue = document.getElementById('glow-value');
        const edgesBtn = document.getElementById('edges-btn');
        const downloadBtn = document.getElementById('download-btn');

        const FRAME_RATE_ASSUMPTION = 30;
        let animationFrameId = null;
        let isGlowEnabled = false;
        let glowAmount = 5;
        let isEdgesEnabled = false;
        let mediaRecorder = null;
        let recordedChunks = [];
        
        // WebGL variables
        let gl, shaderProgram, positionBuffer, videoTexture1, videoTexture2;

        // --- Event Listeners ---
        uploadInput.addEventListener('change', handleFileUpload);
        playPauseBtn.addEventListener('click', togglePlayback);
        offsetSlider.addEventListener('input', handleSliderChange);
        speedSlider.addEventListener('input', handleSpeedChange);
        uploadNewBtn.addEventListener('click', resetForNewUpload);
        glowBtn.addEventListener('click', toggleGlow);
        glowSlider.addEventListener('input', handleGlowSliderChange);
        edgesBtn.addEventListener('click', toggleEdges);
        downloadBtn.addEventListener('click', handleDownload);
        video1.addEventListener('loadedmetadata', setupCanvasAndWebGL);
        
        // --- WebGL Setup ---
        function initWebGL() {
            gl = canvas.getContext('webgl', { preserveDrawingBuffer: true });
            if (!gl) {
                alert('WebGL is not supported by your browser.');
                return false;
            }
            return true;
        }

        function createShader(gl, type, source) {
            const shader = gl.createShader(type);
            gl.shaderSource(shader, source);
            gl.compileShader(shader);
            if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
                console.error('An error occurred compiling the shaders: ' + gl.getShaderInfoLog(shader));
                gl.deleteShader(shader);
                return null;
            }
            return shader;
        }

        function initShaderProgram(gl, vsSource, fsSource) {
            const vertexShader = createShader(gl, gl.VERTEX_SHADER, vsSource);
            const fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fsSource);

            const program = gl.createProgram();
            gl.attachShader(program, vertexShader);
            gl.attachShader(program, fragmentShader);
            gl.linkProgram(program);

            if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
                console.error('Unable to initialize the shader program: ' + gl.getProgramInfoLog(program));
                return null;
            }
            return program;
        }

        function initBuffers() {
            positionBuffer = gl.createBuffer();
            gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
            const positions = [-1.0, 1.0, 1.0, 1.0, -1.0, -1.0, 1.0, -1.0];
            gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);
        }

        function createVideoTexture() {
            const texture = gl.createTexture();
            gl.bindTexture(gl.TEXTURE_2D, texture);
            gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
            gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
            gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
            gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
            return texture;
        }
        
        function updateTexture(texture, video) {
            gl.bindTexture(gl.TEXTURE_2D, texture);
            gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, video);
        }

        // --- Core Functions ---
        function handleDownload() {
             if (mediaRecorder && mediaRecorder.state === 'recording') {
                mediaRecorder.stop();
                video1.removeEventListener('ended', stopRecording);
            } else {
                recordedChunks = [];
                const stream = canvas.captureStream(FRAME_RATE_ASSUMPTION);
                if (typeof MediaRecorder === 'undefined' || !MediaRecorder.isTypeSupported('video/webm; codecs=vp9')) {
                    alert('Your browser does not support video recording. Please try Chrome or Firefox.');
                    return;
                }
                mediaRecorder = new MediaRecorder(stream, { mimeType: 'video/webm; codecs=vp9' });
                mediaRecorder.ondataavailable = event => { if (event.data.size > 0) recordedChunks.push(event.data); };
                mediaRecorder.onstop = () => {
                    const blob = new Blob(recordedChunks, { type: 'video/webm' });
                    const url = URL.createObjectURL(blob);
                    const a = document.createElement('a');
                    a.href = url;
                    a.download = 'motion-extraction.webm';
                    document.body.appendChild(a);
                    a.click();
                    setTimeout(() => {
                        document.body.removeChild(a);
                        URL.revokeObjectURL(url);
                    }, 100);
                    downloadBtn.textContent = 'Download';
                    downloadBtn.classList.remove('btn-red');
                    downloadBtn.classList.add('btn-green');
                    setControlsDisabled(false);
                    video1.loop = true;
                };
                setControlsDisabled(true);
                downloadBtn.disabled = false;
                downloadBtn.textContent = 'Stop & Save';
                downloadBtn.classList.remove('btn-green');
                downloadBtn.classList.add('btn-red');
                video1.loop = false;
                video1.currentTime = 0;
                handleSliderChange();
                mediaRecorder.start();
                if (video1.paused) togglePlayback();
                video1.addEventListener('ended', stopRecording, { once: true });
            }
        }
        
        function stopRecording() { if (mediaRecorder && mediaRecorder.state === 'recording') mediaRecorder.stop(); }
        
        function setControlsDisabled(isDisabled) {
            playPauseBtn.disabled = isDisabled;
            edgesBtn.disabled = isDisabled;
            glowBtn.disabled = isDisabled;
            uploadNewBtn.disabled = isDisabled;
            offsetSlider.disabled = isDisabled;
            speedSlider.disabled = isDisabled;
            glowSlider.disabled = !isGlowEnabled || isDisabled;
        }

        function toggleEdges() {
            isEdgesEnabled = !isEdgesEnabled;
            edgesBtn.classList.toggle('btn-gray', !isEdgesEnabled);
            edgesBtn.classList.toggle('btn-indigo', isEdgesEnabled);
        }

        function toggleGlow() {
            isGlowEnabled = !isGlowEnabled;
            glowSlider.disabled = !isGlowEnabled;
            glowBtn.classList.toggle('btn-gray', !isGlowEnabled);
            glowBtn.classList.toggle('btn-indigo', isGlowEnabled);
        }
        
        function handleGlowSliderChange() {
            glowAmount = parseFloat(glowSlider.value);
            glowValue.textContent = glowAmount;
        }

        function handleFileUpload(event) {
            const file = event.target.files[0];
            if (file && file.type === 'video/mp4') {
                const videoURL = URL.createObjectURL(file);
                video1.src = videoURL;
                video2.src = videoURL;
                speedSlider.value = 1;
                handleSpeedChange();
                uploadContainer.classList.add('hidden');
                playerContainer.classList.remove('hidden');
            } else {
                alert('Please upload a valid MP4 file.');
            }
        }
        
        function setupCanvasAndWebGL() {
            canvas.width = video1.videoWidth;
            canvas.height = video1.videoHeight;
            
            const canvasContainer = canvas.parentElement;
            if (video1.videoHeight > 0) {
                const aspectRatio = video1.videoWidth / video1.videoHeight;
                canvasContainer.style.aspectRatio = `${aspectRatio}`;
            }

            if (!initWebGL()) return;
            
            const vsSource = document.getElementById('vertex-shader').text;
            const fsSource = document.getElementById('fragment-shader').text;
            shaderProgram = initShaderProgram(gl, vsSource, fsSource);
            
            initBuffers();
            videoTexture1 = createVideoTexture();
            videoTexture2 = createVideoTexture();
            
            gl.useProgram(shaderProgram);
            handleSliderChange();
        }

        function togglePlayback() {
            if (video1.paused) {
                handleSliderChange(); 
                video1.play();
                video2.play();
                playPauseBtn.textContent = 'Pause';
                renderLoop();
            } else {
                video1.pause();
                video2.pause();
                playPauseBtn.textContent = 'Play';
                if(animationFrameId) {
                    cancelAnimationFrame(animationFrameId);
                    animationFrameId = null;
                }
            }
        }

        function handleSliderChange() {
            const offsetFrames = parseInt(offsetSlider.value);
            offsetValue.textContent = `${offsetFrames}`;
            const offsetSeconds = offsetFrames / FRAME_RATE_ASSUMPTION;
            if (video1.duration) {
                 video2.currentTime = (video1.currentTime + offsetSeconds) % video1.duration;
            }
        }
        
        function handleSpeedChange() {
            const speed = parseFloat(speedSlider.value);
            video1.playbackRate = speed;
            video2.playbackRate = speed;
            speedValue.textContent = speed.toFixed(2);
        }

        function resetForNewUpload() {
            video1.pause();
            video2.pause();
            if (animationFrameId) cancelAnimationFrame(animationFrameId);
            if (video1.src) URL.revokeObjectURL(video1.src);
            video1.src = '';
            video2.src = '';
            uploadInput.value = '';
            uploadContainer.classList.remove('hidden');
            playerContainer.classList.add('hidden');
            playPauseBtn.textContent = 'Play';
            offsetSlider.value = 50;
            offsetValue.textContent = '50';
            speedSlider.value = 1;
            speedValue.textContent = '1.00';
            isGlowEnabled = false;
            glowBtn.classList.remove('btn-indigo');
            glowBtn.classList.add('btn-gray');
            glowSlider.value = 5;
            glowValue.textContent = '5';
            glowAmount = 5;
            glowSlider.disabled = true;
            isEdgesEnabled = false;
            edgesBtn.classList.remove('btn-indigo');
            edgesBtn.classList.add('btn-gray');
            
            const canvasContainer = canvas.parentElement;
            canvasContainer.style.aspectRatio = '';
        }

        function renderLoop() {
            if (video1.paused || video1.ended) {
                return;
            }

            updateTexture(videoTexture1, video1);
            updateTexture(videoTexture2, video2);

            const resolutionLocation = gl.getUniformLocation(shaderProgram, "u_resolution");
            gl.uniform2f(resolutionLocation, canvas.width, canvas.height);

            const video1Location = gl.getUniformLocation(shaderProgram, "u_video1");
            gl.uniform1i(video1Location, 0);
            
            const video2Location = gl.getUniformLocation(shaderProgram, "u_video2");
            gl.uniform1i(video2Location, 1);

            const edgesLocation = gl.getUniformLocation(shaderProgram, "u_edges");
            gl.uniform1i(edgesLocation, isEdgesEnabled);

            const glowLocation = gl.getUniformLocation(shaderProgram, "u_glow");
            gl.uniform1i(glowLocation, isGlowEnabled);
            
            const glowAmountLocation = gl.getUniformLocation(shaderProgram, "u_glowAmount");
            gl.uniform1f(glowAmountLocation, glowAmount);

            gl.activeTexture(gl.TEXTURE0);
            gl.bindTexture(gl.TEXTURE_2D, videoTexture1);
            gl.activeTexture(gl.TEXTURE1);
            gl.bindTexture(gl.TEXTURE_2D, videoTexture2);

            const positionLocation = gl.getAttribLocation(shaderProgram, "a_position");
            gl.enableVertexAttribArray(positionLocation);
            gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
            gl.vertexAttribPointer(positionLocation, 2, gl.FLOAT, false, 0, 0);

            gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);

            animationFrameId = requestAnimationFrame(renderLoop);
        }
    });
    </script>
</body>
</html>

