<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Instant Gamble Scanner | 2026 Player Protection</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap" rel="stylesheet">

    <style>
        body { font-family: 'Inter', sans-serif; background-color: #0f172a; color: #f8fafc; }
        .gauge-container { position: relative; width: 200px; height: 100px; overflow: hidden; margin: 0 auto; }
        .gauge-bg { width: 200px; height: 200px; border-radius: 50%; border: 20px solid #334155; border-bottom: 0; box-sizing: border-box; }
        .gauge-fill { position: absolute; top: 0; left: 0; width: 200px; height: 200px; border-radius: 50%; border: 20px solid transparent; border-top-color: #10b981; border-right-color: #f59e0b; border-bottom-color: transparent; border-left-color: #ef4444; transform: rotate(-45deg); z-index: 1; opacity: 0.3; }
        .needle { position: absolute; bottom: 0; left: 100px; width: 4px; height: 90px; background: #fff; transform-origin: bottom center; transform: rotate(-90deg); transition: transform 0.5s cubic-bezier(0.4, 0.0, 0.2, 1); z-index: 10; border-radius: 2px; }
        .needle::after { content: ''; position: absolute; bottom: -5px; left: -3px; width: 10px; height: 10px; background: #fff; border-radius: 50%; }
        
        .btn-option { transition: all 0.2s; }
        .btn-option:active { transform: scale(0.98); }
        .fade-in { animation: fadeIn 0.5s ease-in-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body class="flex flex-col min-h-screen">

    <header class="p-4 border-b border-slate-700 flex justify-between items-center bg-slate-900 sticky top-0 z-50">
        <div class="flex items-center gap-2">
            <i data-lucide="scan-line" class="text-emerald-400"></i>
            <span class="font-bold text-lg tracking-tight">GambleScanner</span>
        </div>
        <div class="text-xs bg-slate-800 px-2 py-1 rounded-full text-slate-400" id="referral-badge">
            Points: <span id="points-display" class="text-emerald-400 font-bold">0</span>
        </div>
    </header>

    <div class="w-full h-1 bg-slate-800">
        <div id="progress-bar" class="h-full bg-emerald-500 transition-all duration-300" style="width: 25%"></div>
    </div>

    <main class="flex-grow p-6 max-w-md mx-auto w-full fade-in">
        
        <div class="text-center mb-8">
            <h2 class="text-sm uppercase tracking-widest text-slate-400 mb-4">Risk Monitor</h2>
            <div class="gauge-container">
                <div class="gauge-bg"></div>
                <div class="gauge-fill"></div>
                <div class="needle" id="gauge-needle"></div>
            </div>
            <p id="risk-status" class="mt-2 font-bold text-emerald-400 text-lg">Analysis Pending...</p>
        </div>

        <div id="app-content">
            <div id="step-basic">
                <h1 class="text-2xl font-bold mb-2">Basic Markers of Harm</h1>
                <p class="text-slate-400 text-sm mb-6">Anonymous pre-check. Is your play still safe?</p>
                
                <div id="question-container" class="space-y-4">
                    </div>
            </div>
        </div>

    </main>

    <footer class="bg-slate-900 border-t border-slate-800 p-6 mt-auto">
        <div class="max-w-md mx-auto text-center">
            <p class="text-slate-500 text-xs mb-2">Region Detected: <span id="region-display" class="text-slate-300 font-semibold">Global</span></p>
            <div id="helpline-box" class="bg-slate-800 rounded p-3 mb-4 hidden">
                <p class="text-xs text-slate-400">Need immediate help?</p>
                <a href="#" id="helpline-number" class="text-emerald-400 font-bold text-sm">Call Generic Support</a>
            </div>
            <p class="text-[10px] text-slate-600">
                © 2026 GambleScanner. Compliant with GDPR, UKGC Markers of Harm, and NCPG Standards. 
                Data is processed locally on your device.
            </p>
        </div>
    </footer>

    <script>
        // --- 1. CONFIGURATION & DATA ---
        const questions = [
            { id: 'q1', text: "Do you feel bad or stressed after gambling?" },
            { id: 'q2', text: "Is gambling making it hard for you to pay bills?" }, // Affordability Check
            { id: 'q3', text: "Do you bet more to win back what you lost?" },       // Chase Betting
            { id: 'q4', text: "Are you in any betting groups on social media?" },
            { id: 'q5', text: "Do you get alerts for betting or live scores?" }
        ];

        let state = {
            step: 0,
            currentQuestionIndex: 0,
            yesCount: 0,
            points: 0,
            region: 'UK', // Default, would be detected via IP in real app
            answers: []
        };

        // --- 2. DOM ELEMENTS ---
        const contentDiv = document.getElementById('app-content');
        const needle = document.getElementById('gauge-needle');
        const riskText = document.getElementById('risk-status');
        const pointsDisplay = document.getElementById('points-display');
        const progressBar = document.getElementById('progress-bar');
        const regionDisplay = document.getElementById('region-display');
        const helplineBox = document.getElementById('helpline-box');
        const helplineNum = document.getElementById('helpline-number');

        // --- 3. CORE LOGIC ---

        function init() {
            lucide.createIcons();
            detectRegion();
            renderQuestion();
        }

        function detectRegion() {
            // Simulating region detection logic
            // In a real app, use an API or browser locale
            const timeZone = Intl.DateTimeFormat().resolvedOptions().timeZone;
            if (timeZone.includes('London')) state.region = 'UK';
            else if (timeZone.includes('New_York')) state.region = 'USA';
            else if (timeZone.includes('Sydney')) state.region = 'AUS';
            
            regionDisplay.innerText = state.region;
            updateFooterCompliance();
        }

        function updateFooterCompliance() {
            helplineBox.classList.remove('hidden');
            if(state.region === 'UK') {
                helplineNum.innerText = "Call GamCare: 0808 8020 133";
                helplineNum.href = "tel:08088020133";
            } else if (state.region === 'USA') {
                helplineNum.innerText = "Call 1-800-GAMBLER";
                helplineNum.href = "tel:18004262537";
            } else if (state.region === 'AUS') {
                helplineNum.innerText = "Gambling Help: 1800 858 858";
            } else {
                helplineNum.innerText = "Find Local Support";
            }
        }

        function updateGauge() {
            // Rotate needle based on Yes count (0 to 5)
            // 5 questions. Max rotation 90deg (Red). Min -90deg (Green).
            // Let's map 0-5 'yes' to rotation.
            const percentage = state.yesCount / 5;
            const degrees = -90 + (percentage * 180); 
            needle.style.transform = `rotate(${degrees}deg)`;

            // Update Text Color
            if(state.yesCount <= 1) {
                riskText.innerText = "Balanced Play";
                riskText.className = "mt-2 font-bold text-emerald-400 text-lg";
            } else if (state.yesCount <= 3) {
                riskText.innerText = "Caution Advised";
                riskText.className = "mt-2 font-bold text-amber-400 text-lg";
            } else {
                riskText.innerText = "High Risk Detected";
                riskText.className = "mt-2 font-bold text-red-500 text-lg";
            }
        }

        function renderQuestion() {
            if (state.currentQuestionIndex < questions.length) {
                const q = questions[state.currentQuestionIndex];
                contentDiv.innerHTML = `
                    <div class="fade-in">
                        <div class="mb-2 text-xs text-slate-500 uppercase font-bold">Question ${state.currentQuestionIndex + 1} of 5</div>
                        <h2 class="text-xl font-semibold mb-6">${q.text}</h2>
                        <div class="grid grid-cols-2 gap-4">
                            <button onclick="handleAnswer(true)" class="btn-option bg-slate-800 hover:bg-slate-700 border border-slate-600 p-4 rounded-xl flex flex-col items-center gap-2">
                                <i data-lucide="check-circle" class="text-emerald-500"></i>
                                <span class="font-bold">Yes</span>
                            </button>
                            <button onclick="handleAnswer(false)" class="btn-option bg-slate-800 hover:bg-slate-700 border border-slate-600 p-4 rounded-xl flex flex-col items-center gap-2">
                                <i data-lucide="x-circle" class="text-slate-400"></i>
                                <span class="font-bold">No</span>
                            </button>
                        </div>
                    </div>
                `;
                lucide.createIcons();
            } else {
                showBasicResults();
            }
        }

        function handleAnswer(isYes) {
            if (isYes) state.yesCount++;
            state.answers.push(isYes);
            state.currentQuestionIndex++;
            
            // Update UI
            updateGauge();
            let progress = 25 + ((state.currentQuestionIndex / 5) * 25); // Move bar
            progressBar.style.width = `${progress}%`;
            
            renderQuestion();
        }

        function showBasicResults() {
            // Logic Split
            if (state.yesCount < 2) {
                // Low Risk Flow -> Referral
                contentDiv.innerHTML = `
                    <div class="fade-in text-center">
                        <div class="w-16 h-16 bg-emerald-500/20 rounded-full flex items-center justify-center mx-auto mb-4">
                            <i data-lucide="shield-check" class="text-emerald-400 w-8 h-8"></i>
                        </div>
                        <h2 class="text-2xl font-bold mb-2">You're in Control</h2>
                        <p class="text-slate-400 mb-6">Your results show healthy play habits. Keep it that way by helping others.</p>
                        
                        <div class="bg-slate-800 p-4 rounded-xl mb-6 border border-slate-700">
                            <h3 class="font-bold text-white mb-2">Referral Challenge</h3>
                            <p class="text-xs text-slate-400 mb-4">Share this scanner with 5 friends to unlock the "Safe Play Badge".</p>
                            <button onclick="referFriend()" class="w-full bg-emerald-600 hover:bg-emerald-500 text-white font-bold py-3 rounded-lg flex items-center justify-center gap-2">
                                <i data-lucide="share-2"></i> Share & Earn (+10 Pts)
                            </button>
                        </div>
                        
                        <button onclick="startStandardCheck()" class="text-sm text-slate-500 underline">Continue to Demographic Check</button>
                    </div>
                `;
            } else {
                // High Risk Flow -> Standard/Paid
                contentDiv.innerHTML = `
                    <div class="fade-in text-center">
                        <div class="w-16 h-16 bg-red-500/20 rounded-full flex items-center justify-center mx-auto mb-4">
                            <i data-lucide="alert-triangle" class="text-red-500 w-8 h-8"></i>
                        </div>
                        <h2 class="text-2xl font-bold mb-2">Markers of Harm Detected</h2>
                        <p class="text-slate-400 mb-6">Your answers suggest you are entering a high-stress zone.</p>
                        
                        <button onclick="startStandardCheck()" class="w-full bg-gradient-to-r from-red-600 to-red-500 hover:from-red-500 hover:to-red-400 text-white font-bold py-3 rounded-lg mb-4 shadow-lg shadow-red-900/50">
                            FIND OUT MORE <i data-lucide="arrow-right" class="inline w-4 h-4 ml-1"></i>
                        </button>
                        <p class="text-xs text-slate-500">Unlocking your Standard Profile helps us benchmark your data anonymously.</p>
                    </div>
                `;
            }
            lucide.createIcons();
        }

        function referFriend() {
            state.points += 10;
            pointsDisplay.innerText = state.points;
            alert("Mock Referral Link Copied! (In real app, opens native share sheet)");
        }

        function startStandardCheck() {
            progressBar.style.width = "50%";
            contentDiv.innerHTML = `
                <div class="fade-in">
                    <h2 class="text-xl font-bold mb-2">Standard Profile</h2>
                    <p class="text-slate-400 text-sm mb-6">Compare your habits to the 2026 ${state.region} Average.</p>
                    
                    <form onsubmit="handleStandardSubmit(event)" class="space-y-4">
                        <div>
                            <label class="block text-xs text-slate-500 mb-1">Age Bracket</label>
                            <select class="w-full bg-slate-800 border border-slate-700 rounded p-3 text-white">
                                <option>18-24 (High Risk Group)</option>
                                <option>25-34</option>
                                <option>35-50</option>
                                <option>50+</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-xs text-slate-500 mb-1">Gender</label>
                            <select class="w-full bg-slate-800 border border-slate-700 rounded p-3 text-white">
                                <option>Male</option>
                                <option>Female</option>
                                <option>Prefer not to say</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-xs text-slate-500 mb-1">Email (For Full Report)</label>
                            <input type="email" placeholder="you@example.com" class="w-full bg-slate-800 border border-slate-700 rounded p-3 text-white">
                        </div>
                        <button type="submit" class="w-full bg-emerald-600 hover:bg-emerald-500 text-white font-bold py-3 rounded-lg mt-4">
                            Analyze Profile
                        </button>
                    </form>
                </div>
            `;
        }

        function handleStandardSubmit(e) {
            e.preventDefault();
            showAdvancedPromo();
        }

        function showAdvancedPromo() {
            // This is where we upsell the Paid/Advanced Service
            progressBar.style.width = "100%";
            contentDiv.innerHTML = `
                <div class="fade-in text-center">
                    <div class="mb-6">
                        <span class="bg-yellow-500/20 text-yellow-500 text-xs font-bold px-2 py-1 rounded">ENHANCED CHECK RECOMMENDED</span>
                    </div>
                    <h2 class="text-2xl font-bold mb-2">Your Profile is Ready</h2>
                    <p class="text-slate-400 mb-6">Based on your demographics, you are in a category frequently targeted by betting algorithms.</p>
                    
                    <div class="bg-slate-800 border border-slate-700 p-6 rounded-xl mb-4 text-left relative overflow-hidden">
                        <div class="absolute top-0 right-0 bg-yellow-500 text-black text-[10px] font-bold px-2 py-1">PREMIUM</div>
                        <h3 class="font-bold text-white text-lg">The Total Reset Bundle</h3>
                        <ul class="text-sm text-slate-400 mt-2 space-y-2">
                            <li class="flex items-center gap-2"><i data-lucide="check" class="w-4 h-4 text-emerald-500"></i> Full Psychology Breakdown</li>
                            <li class="flex items-center gap-2"><i data-lucide="check" class="w-4 h-4 text-emerald-500"></i> "Exit Strategy" E-Book</li>
                            <li class="flex items-center gap-2"><i data-lucide="check" class="w-4 h-4 text-emerald-500"></i> 1 Therapeutic Conversation</li>
                        </ul>
                        <button class="w-full bg-white text-slate-900 font-bold py-3 rounded-lg mt-4 hover:bg-slate-200">
                            Get Access ($19.99)
                        </button>
                    </div>

                    <p class="text-xs text-slate-500 mt-4">Or <a href="#" class="underline text-emerald-400">Book a Free 15-min Discovery Call</a></p>
                </div>
            `;
            lucide.createIcons();
        }

        // Start App
        init();

    </script>
</body>
</html>
