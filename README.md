<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HomeschoolHero | Pro Tracker</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .gradient-text { background: linear-gradient(to right, #6366f1, #a855f7); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .hidden { display: none; }
        .progress-bar { transition: width 1s cubic-bezier(0.4, 0, 0.2, 1); }
    </style>
</head>
<body class="bg-slate-50 font-sans antialiased">

    <div id="salesPage" class="min-h-screen">
        <nav class="p-6 flex justify-between items-center max-w-7xl mx-auto">
            <div class="text-2xl font-bold text-indigo-600">🏠 HomeschoolHero</div>
            <button onclick="showPage('appDashboard')" class="bg-indigo-600 text-white px-6 py-2 rounded-full font-bold shadow-md">Open App</button>
        </nav>
        <header class="max-w-4xl mx-auto text-center py-20 px-6">
            <h1 class="text-5xl md:text-6xl font-black mb-6 leading-tight text-slate-900">Education is <span class="gradient-text">Life.</span></h1>
            <p class="text-xl text-slate-600 mb-10">Turn every gaming session, meal prep, and chore into a professional academic transcript.</p>
            <button onclick="showPage('appDashboard')" class="bg-slate-900 text-white px-10 py-4 rounded-full font-bold text-lg hover:bg-black transition">Launch My Console</button>
        </header>
    </div>

    <div id="appDashboard" class="hidden min-h-screen">
        <nav class="bg-indigo-600 text-white p-4 sticky top-0 z-50">
            <div class="max-w-6xl mx-auto flex justify-between items-center">
                <span class="font-bold flex items-center gap-2">🏠 HomeschoolHero <span class="text-[10px] bg-indigo-500 px-2 py-0.5 rounded">BETA</span></span>
                <div class="flex gap-2">
                    <button onclick="exportToCSV()" class="text-xs bg-green-500 px-3 py-1 rounded font-bold">Export CSV</button>
                    <button onclick="clearAllData()" class="text-xs bg-red-500 px-3 py-1 rounded">Reset</button>
                </div>
            </div>
        </nav>
        
        <main class="max-w-5xl mx-auto p-6 md:p-8">
            <div class="bg-white p-6 md:p-8 rounded-3xl shadow-sm border border-slate-200 mb-8">
                <h2 class="text-xl font-bold mb-4 text-slate-800">Log an Activity</h2>
                <textarea id="userInput" class="w-full h-32 p-4 border border-slate-200 rounded-2xl mb-4 text-lg outline-none focus:ring-2 focus:ring-indigo-500 transition-all" placeholder="Tell me what happened today... (e.g., 'We baked bread and calculated the cost of yeast')"></textarea>
                <button onclick="processActivity()" class="w-full md:w-auto bg-indigo-600 text-white px-10 py-3 rounded-xl font-bold shadow-lg hover:bg-indigo-700 transition">Analyze & Save</button>
            </div>

            <div id="subjectGrid" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-4 mb-10">
                </div>

            <div class="bg-white p-6 rounded-3xl border border-slate-200 shadow-sm">
                <h3 class="font-bold text-lg mb-6 flex justify-between items-center">
                    Detailed Activity Log
                    <span class="text-sm font-normal text-slate-400">Records stored on device</span>
                </h3>
                <div id="logHistory" class="space-y-4">
                    </div>
            </div>
        </main>
    </div>

    <script>
        const defaultSubjects = {
            ela: { name: "Language Arts", color: "bg-blue-500", hours: 0, target: 120, keywords: ["read", "write", "spelled", "grammar", "book"] },
            math: { name: "Mathematics", color: "bg-green-500", hours: 0, target: 120, keywords: ["measured", "calculated", "budget", "ratio"] },
            science: { name: "Science", color: "bg-yellow-500", hours: 0, target: 120, keywords: ["yeast", "reaction", "nature", "experiment"] },
            history: { name: "Social Studies", color: "bg-red-500", hours: 0, target: 120, keywords: ["past", "map", "culture", "government"] },
            fin: { name: "Financial Lit", color: "bg-emerald-500", hours: 0, target: 60, keywords: ["cost", "tax", "price", "savings"] },
            ent: { name: "Entrepreneur", color: "bg-purple-500", hours: 0, target: 60, keywords: ["product", "marketing", "business", "brand"] },
            lang: { name: "World Language", color: "bg-pink-500", hours: 0, target: 120, keywords: ["spanish", "translation", "phrases"] },
            arts: { name: "The Arts", color: "bg-orange-500", hours: 0, target: 60, keywords: ["sketch", "design", "music", "art"] },
            pe: { name: "Physical Ed", color: "bg-cyan-500", hours: 0, target: 60, keywords: ["exercise", "hiking", "sport", "cleaning"] },
            tech: { name: "Technology", color: "bg-slate-700", hours: 0, target: 60, keywords: ["gaming", "coding", "software", "digital"] }
        };

        let userProgress = JSON.parse(localStorage.getItem('hsHeroProgress')) || defaultSubjects;
        let historyLogs = JSON.parse(localStorage.getItem('hsHeroLogs')) || [];

        function showPage(pageId) {
            document.getElementById('salesPage').classList.add('hidden');
            document.getElementById('appDashboard').classList.add('hidden');
            document.getElementById(pageId).classList.remove('hidden');
            if(pageId === 'appDashboard') renderApp();
        }

        function renderApp() {
            const grid = document.getElementById('subjectGrid');
            grid.innerHTML = '';
            for (let key in userProgress) {
                const sub = userProgress[key];
                const perc = Math.min(100, (sub.hours / sub.target) * 100);
                grid.innerHTML += `
                    <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm hover:border-indigo-300 transition-colors">
                        <div class="flex justify-between items-start mb-2">
                            <h3 class="text-[10px] font-bold text-slate-400 uppercase tracking-widest">${sub.name}</h3>
                            <button onclick="editGoal('${key}')" class="text-[10px] text-indigo-500 hover:underline">Edit Goal</button>
                        </div>
                        <div class="text-2xl font-black text-slate-800 mb-2">${sub.hours.toFixed(1)}<span class="text-xs font-normal text-slate-400"> / ${sub.target}h</span></div>
                        <div class="w-full bg-slate-100 h-2 rounded-full overflow-hidden">
                            <div class="progress-bar ${sub.color} h-full" style="width: ${perc}%"></div>
                        </div>
                    </div>`;
            }

            const historyDiv = document.getElementById('logHistory');
            historyDiv.innerHTML = historyLogs.map((log, index) => `
                <div class="p-4 bg-slate-50 rounded-2xl border border-slate-100 relative group">
                    <div class="flex justify-between items-start mb-2">
                        <span class="text-[10px] font-bold text-slate-400 uppercase">${log.date}</span>
                        <button onclick="deleteLog(${index})" class="text-red-400 opacity-0 group-hover:opacity-100 transition-opacity text-xs">Delete</button>
                    </div>
                    <p class="text-slate-700 leading-relaxed font-medium">"${log.text}"</p>
                    <div class="mt-3 flex flex-wrap gap-2">
                        ${log.tags.map(tag => `<span class="text-[9px] bg-white border border-slate-200 px-2 py-0.5 rounded text-slate-500 font-bold">${tag}</span>`).join('')}
                    </div>
                </div>
            `).join('') || '<div class="text-center py-10 text-slate-400">No activities logged yet. Share your first success story above!</div>';
        }

        function processActivity() {
            const input = document.getElementById('userInput');
            const text = input.value.trim().toLowerCase();
            if (!text) return;

            let matchedTags = [];
            for (let key in userProgress) {
                userProgress[key].keywords.forEach(word => {
                    if (text.includes(word)) {
                        userProgress[key].hours += 0.5;
                        if (!matchedTags.includes(userProgress[key].name)) matchedTags.push(userProgress[key].name);
                    }
                });
            }

            if(matchedTags.length > 0) {
                historyLogs.unshift({ 
                    date: new Date().toLocaleDateString() + ' ' + new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}), 
                    text: input.value,
                    tags: matchedTags 
                });
                saveData();
                renderApp();
                input.value = '';
            } else {
                alert("Could not identify specific subjects. Try using more descriptive keywords!");
            }
        }

        function editGoal(key) {
            const newGoal = prompt(`Set target hours for ${userProgress[key].name}:`, userProgress[key].target);
            if (newGoal && !isNaN(newGoal)) {
                userProgress[key].target = parseInt(newGoal);
                saveData();
                renderApp();
            }
        }

        function deleteLog(index) {
            if(confirm("Delete this entry? (Progress hours will remain)")) {
                historyLogs.splice(index, 1);
                saveData();
                renderApp();
            }
        }

        function saveData() {
            localStorage.setItem('hsHeroProgress', JSON.stringify(userProgress));
            localStorage.setItem('hsHeroLogs', JSON.stringify(historyLogs));
        }

        function exportToCSV() {
            let csvContent = "data:text/csv;charset=utf-8,Date,Activity,Subjects\n";
            historyLogs.forEach(log => {
                csvContent += `"${log.date}","${log.text.replace(/"/g, '""')}","${log.tags.join('; ')}"\n`;
            });
            window.open(encodeURI(csvContent));
        }

        function clearAllData() {
            if(confirm("This will permanently delete all progress and logs on this device. Continue?")) {
                localStorage.clear();
                location.reload();
            }
        }

        // Auto-load app if data exists
        if(localStorage.getItem('hsHeroProgress')) {
            showPage('appDashboard');
        }
    </script>
</body>
</html>
