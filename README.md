# Visualizer<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Resonant Frequency Visualizer</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(135deg, #0a0a0a, #1a1a2e, #16213e);
            color: #fff;
            overflow: hidden;
            height: 100vh;
        }
        
        .container {
            display: flex;
            flex-direction: column;
            height: 100vh;
            padding: 20px;
        }
        
        .header {
            text-align: center;
            margin-bottom: 20px;
        }
        
        h1 {
            font-size: 2.5rem;
            background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            margin-bottom: 10px;
        }
        
        .controls {
            display: flex;
            justify-content: center;
            gap: 15px;
            margin-bottom: 20px;
            flex-wrap: wrap;
        }
        
        button {
            padding: 12px 24px;
            border: none;
            border-radius: 25px;
            background: linear-gradient(45deg, #667eea, #764ba2);
            color: white;
            cursor: pointer;
            font-size: 16px;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
        }
        
        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
        }
        
        button:disabled {
            opacity: 0.6;
            cursor: not-allowed;
        }
        
        .visualizer-container {
            flex: 1;
            display: flex;
            gap: 20px;
        }
        
        .canvas-container {
            flex: 1;
            position: relative;
            border-radius: 15px;
            overflow: hidden;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
        }
        
        canvas {
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, #1a1a2e, #0a0a0a);
        }
        
        .frequency-info {
            width: 300px;
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border-radius: 15px;
            padding: 20px;
            border: 1px solid rgba(255, 255, 255, 0.2);
        }
        
        .frequency-display {
            text-align: center;
            margin-bottom: 20px;
        }
        
        .frequency-value {
            font-size: 2rem;
            font-weight: bold;
            color: #4ecdc4;
            margin: 10px 0;
        }
        
        .octave-info {
            background: rgba(0, 0, 0, 0.3);
            border-radius: 10px;
            padding: 15px;
            margin: 10px 0;
        }
        
        .slider-container {
            margin: 15px 0;
        }
        
        .slider {
            width: 100%;
            -webkit-appearance: none;
            height: 6px;
            border-radius: 3px;
            background: rgba(255, 255, 255, 0.3);
            outline: none;
        }
        
        .slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
            cursor: pointer;
        }
        
        .peak-list {
            max-height: 200px;
            overflow-y: auto;
            margin-top: 15px;
        }
        
        .peak-item {
            background: rgba(255, 255, 255, 0.1);
            margin: 5px 0;
            padding: 8px;
            border-radius: 5px;
            font-size: 0.9rem;
        }
        
        .status {
            position: absolute;
            top: 10px;
            right: 10px;
            background: rgba(0, 0, 0, 0.7);
            padding: 8px 15px;
            border-radius: 20px;
            font-size: 0.9rem;
        }
        
        .recording {
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Resonant Frequency Visualizer</h1>
            <p>Detect resonant frequencies and generate 11-octave harmonics</p>
        </div>
        
        <div class="controls">
            <button id="startBtn">Start Microphone</button>
            <button id="stopBtn" disabled>Stop</button>
            <button id="generateBtn" disabled>Generate 11 Octaves Higher</button>
            <button id="playBtn" disabled>Play Generated Tone</button>
            <button id="stopToneBtn" disabled>Stop Tone</button>
        </div>
        
        <div class="visualizer-container">
            <div class="canvas-container">
                <canvas id="visualizer"></canvas>
                <div class="status" id="status">Ready</div>
            </div>
            
            <div class="frequency-info">
                <div class="frequency-display">
                    <h3>Detected Resonant Frequency</h3>
                    <div class="frequency-value" id="resonantFreq">--- Hz</div>
                    <div>Amplitude: <span id="amplitude">---</span></div>
                </div>
                
                <div class="octave-info">
                    <h4>11 Octaves Higher</h4>
                    <div class="frequency-value" id="elevenOctaves">--- Hz</div>
                    <div>Multiplier: 2048x</div>
                </div>
                
                <div class="slider-container">
                    <label>Sensitivity</label>
                    <input type="range" id="sensitivity" class="slider" min="0.01" max="1" step="0.01" value="0.3">
                </div>
                
                <div class="slider-container">
                    <label>Generated Volume</label>
                    <input type="range" id="volume" class="slider" min="0" max="1" step="0.01" value="0.1">
                </div>
                
                <div class="peak-list">
                    <h4>Peak Frequencies</h4>
                    <div id="peaksList"></div>
                </div>
            </div>
        </div>
    </div>

    <script>
        class ResonantFrequencyVisualizer {
            constructor() {
                this.audioContext = null;
                this.microphone = null;
                this.analyser = null;
                this.oscillator = null;
                this.gainNode = null;
                this.dataArray = null;
                this.canvas = document.getElementById('visualizer');
                this.ctx = this.canvas.getContext('2d');
                this.isRecording = false;
                this.resonantFreq = 0;
                this.peaks = [];
                
                this.setupCanvas();
                this.bindEvents();
            }
            
            setupCanvas() {
                this.canvas.width = this.canvas.offsetWidth;
                this.canvas.height = this.canvas.offsetHeight;
                
                window.addEventListener('resize', () => {
                    this.canvas.width = this.canvas.offsetWidth;
                    this.canvas.height = this.canvas.offsetHeight;
                });
            }
            
            bindEvents() {
                document.getElementById('startBtn').addEventListener('click', () => this.startRecording());
                document.getElementById('stopBtn').addEventListener('click', () => this.stopRecording());
                document.getElementById('generateBtn').addEventListener('click', () => this.generateElevenOctaves());
                document.getElementById('playBtn').addEventListener('click', () => this.playGeneratedTone());
                document.getElementById('stopToneBtn').addEventListener('click', () => this.stopTone());
                
                document.getElementById('sensitivity').addEventListener('input', (e) => {
                    this.sensitivity = parseFloat(e.target.value);
                });
                
                document.getElementById('volume').addEventListener('input', (e) => {
                    if (this.gainNode) {
                        this.gainNode.gain.value = parseFloat(e.target.value);
                    }
                });
            }
            
            async startRecording() {
                try {
                    this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
                    
                    const stream = await navigator.mediaDevices.getUserMedia({ 
                        audio: { 
                            echoCancellation: false,
                            noiseSuppression: false,
                            autoGainControl: false
                        } 
                    });
                    
                    this.microphone = this.audioContext.createMediaStreamSource(stream);
                    this.analyser = this.audioContext.createAnalyser();
                    
                    this.analyser.fftSize = 8192;
                    this.analyser.smoothingTimeConstant = 0.3;
                    
                    this.microphone.connect(this.analyser);
                    
                    this.dataArray = new Uint8Array(this.analyser.frequencyBinCount);
                    this.freqArray = new Float32Array(this.analyser.frequencyBinCount);
                    
                    this.isRecording = true;
                    this.sensitivity = 0.3;
                    
                    document.getElementById('startBtn').disabled = true;
                    document.getElementById('stopBtn').disabled = false;
                    document.getElementById('generateBtn').disabled = false;
                    document.getElementById('status').textContent = 'Recording';
                    document.getElementById('status').classList.add('recording');
                    
                    this.visualize();
                } catch (error) {
                    console.error('Error accessing microphone:', error);
                    document.getElementById('status').textContent = 'Microphone access denied';
                }
            }
            
            stopRecording() {
                this.isRecording = false;
                
                if (this.microphone) {
                    this.microphone.disconnect();
                }
                
                if (this.audioContext) {
                    this.audioContext.close();
                }
                
                document.getElementById('startBtn').disabled = false;
                document.getElementById('stopBtn').disabled = true;
                document.getElementById('generateBtn').disabled = true;
                document.getElementById('status').textContent = 'Stopped';
                document.getElementById('status').classList.remove('recording');
                
                this.stopTone();
            }
            
            visualize() {
                if (!this.isRecording) return;
                
                this.analyser.getByteFrequencyData(this.dataArray);
                this.analyser.getFloatFrequencyData(this.freqArray);
                
                this.detectResonantFrequency();
                this.drawVisualizer();
                
                requestAnimationFrame(() => this.visualize());
            }
            
            detectResonantFrequency() {
                const sampleRate = this.audioContext.sampleRate;
                const nyquist = sampleRate / 2;
                const binSize = nyquist / this.dataArray.length;
                
                let maxAmplitude = 0;
                let resonantBin = 0;
                this.peaks = [];
                
                // Find peaks
                for (let i = 5; i < this.dataArray.length - 5; i++) {
                    const amplitude = this.dataArray[i];
                    const freq = i * binSize;
                    
                    if (amplitude > maxAmplitude && amplitude > 50) {
                        // Check if it's a local maximum
                        let isPeak = true;
                        for (let j = i - 2; j <= i + 2; j++) {
                            if (j !== i && this.dataArray[j] >= amplitude) {
                                isPeak = false;
                                break;
                            }
                        }
                        
                        if (isPeak && amplitude > this.sensitivity * 255) {
                            this.peaks.push({ freq: freq, amplitude: amplitude });
                            
                            if (amplitude > maxAmplitude) {
                                maxAmplitude = amplitude;
                                resonantBin = i;
                                this.resonantFreq = freq;
                            }
                        }
                    }
                }
                
                // Sort peaks by amplitude
                this.peaks.sort((a, b) => b.amplitude - a.amplitude);
                this.peaks = this.peaks.slice(0, 10); // Keep top 10
                
                this.updateDisplay();
            }
            
            updateDisplay() {
                document.getElementById('resonantFreq').textContent = 
                    this.resonantFreq > 0 ? `${this.resonantFreq.toFixed(2)} Hz` : '--- Hz';
                
                document.getElementById('amplitude').textContent = 
                    this.resonantFreq > 0 ? `${(this.peaks[0]?.amplitude / 255 * 100).toFixed(1)}%` : '---';
                
                const elevenOctaves = this.resonantFreq * Math.pow(2, 11);
                document.getElementById('elevenOctaves').textContent = 
                    this.resonantFreq > 0 ? `${elevenOctaves.toFixed(2)} Hz` : '--- Hz';
                
                // Update peaks list
                const peaksList = document.getElementById('peaksList');
                peaksList.innerHTML = this.peaks.map(peak => 
                    `<div class="peak-item">${peak.freq.toFixed(2)} Hz (${(peak.amplitude/255*100).toFixed(1)}%)</div>`
                ).join('');
            }
            
            drawVisualizer() {
                const width = this.canvas.width;
                const height = this.canvas.height;
                
                this.ctx.fillStyle = 'rgba(10, 10, 10, 0.1)';
                this.ctx.fillRect(0, 0, width, height);
                
                // Draw frequency spectrum
                const barWidth = width / this.dataArray.length * 4;
                let x = 0;
                
                for (let i = 0; i < this.dataArray.length / 4; i++) {
                    const barHeight = (this.dataArray[i] / 255) * height * 0.8;
                    
                    // Create gradient based on frequency
                    const hue = (i / this.dataArray.length * 4) * 360;
                    this.ctx.fillStyle = `hsl(${hue}, 70%, 50%)`;
                    
                    // Highlight resonant frequency
                    if (this.peaks.some(peak => Math.abs(peak.freq - (i * this.audioContext.sampleRate / 2 / this.dataArray.length)) < 50)) {
                        this.ctx.fillStyle = '#ff6b6b';
                        this.ctx.shadowColor = '#ff6b6b';
                        this.ctx.shadowBlur = 20;
                    } else {
                        this.ctx.shadowBlur = 0;
                    }
                    
                    this.ctx.fillRect(x, height - barHeight, barWidth - 2, barHeight);
                    x += barWidth;
                }
                
                // Draw waveform
                this.ctx.strokeStyle = '#4ecdc4';
                this.ctx.lineWidth = 2;
                this.ctx.beginPath();
                
                const sliceWidth = width / this.dataArray.length;
                x = 0;
                
                for (let i = 0; i < this.dataArray.length; i++) {
                    const v = this.dataArray[i] / 255;
                    const y = height - (v * height * 0.3);
                    
                    if (i === 0) {
                        this.ctx.moveTo(x, y);
                    } else {
                        this.ctx.lineTo(x, y);
                    }
                    
                    x += sliceWidth;
                }
                
                this.ctx.stroke();
            }
            
            generateElevenOctaves() {
                if (this.resonantFreq > 0) {
                    this.generatedFreq = this.resonantFreq * Math.pow(2, 11); // 2^11 = 2048
                    document.getElementById('playBtn').disabled = false;
                    document.getElementById('status').textContent = `Generated: ${this.generatedFreq.toFixed(2)} Hz`;
                }
            }
            
            async playGeneratedTone() {
                if (!this.generatedFreq) return;
                
                if (!this.audioContext || this.audioContext.state === 'closed') {
                    this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
                }
                
                if (this.audioContext.state === 'suspended') {
                    await this.audioContext.resume();
                }
                
                this.stopTone(); // Stop any existing tone
                
                this.oscillator = this.audioContext.createOscillator();
                this.gainNode = this.audioContext.createGain();
                
                this.oscillator.type = 'sine';
                this.oscillator.frequency.setValueAtTime(this.generatedFreq, this.audioContext.currentTime);
                
                const volume = parseFloat(document.getElementById('volume').value);
                this.gainNode.gain.setValueAtTime(volume, this.audioContext.currentTime);
                
                this.oscillator.connect(this.gainNode);
                this.gainNode.connect(this.audioContext.destination);
                
                this.oscillator.start();
                
                document.getElementById('playBtn').disabled = true;
                document.getElementById('stopToneBtn').disabled = false;
            }
            
            stopTone() {
                if (this.oscillator) {
                    this.oscillator.stop();
                    this.oscillator.disconnect();
                    this.oscillator = null;
                }
                
                if (this.gainNode) {
                    this.gainNode.disconnect();
                    this.gainNode = null;
                }
                
                document.getElementById('playBtn').disabled = false;
                document.getElementById('stopToneBtn').disabled = true;
            }
        }
        
        // Initialize the visualizer
        const visualizer = new ResonantFrequencyVisualizer();
    </script>
</body>
</html>
