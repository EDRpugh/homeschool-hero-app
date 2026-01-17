<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HomeschoolHero | Persistent Tracker</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .gradient-text { background: linear-gradient(to right, #6366f1, #a855f7); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .hidden { display: none; }
        .progress-bar { transition: width 0.8s cubic-bezier(0.4, 0, 0.2, 1); }
    </style>
</head>
<body class="bg-slate-50 font-sans antialiased">

    <div id="salesPage" class="min-h-screen">
        <nav class="p-6 flex justify-between items-center max-w-7xl mx-auto">
            <div class="text-2xl font-bold text-indigo-600">🏠 HomeschoolHero</div>
            <button onclick="showPage('loginPage')" class="text-gray-600 font-semibold hover:text-indigo-600">Login</button>
        </nav>

        <header class="max-w-4xl mx-auto text-center py-20 px-6">
            <h1 class="text-5xl md:text-6xl font-black mb-6 leading-tight text-slate-900">
                Stop Chasing Worksheets. <br>
                <span class="gradient-text">Validate Real Life.</span>
            </h1>
            <p class="text-xl text-slate-600 mb-10 leading-relaxed">
                Society says "education" happens at a desk. We prove it happens everywhere. Turn gaming, cooking, and chores into an accredited transcript.
            </p>
            <div class="flex flex-col md:flex-row gap-4 justify-center">
                <button onclick="showPage('loginPage')" class="bg-indigo-600 text-white px-10 py-4 rounded-full font-bold text-lg shadow-lg hover:bg-indigo-700 transition">Get Started Free</button>
            </div>
        </header>
    </div>

    <div id="loginPage" class="hidden min-h-screen flex items-center justify-center p-6 bg-indigo-50">
        <div class="bg-white p-10 rounded-3xl shadow-2xl w-full max-w-md text-center">
            <h2 class="text-3xl font-bold mb-2">Welcome Back</h2>
            <p class="text-slate-500 mb-8">Data is stored locally on this device.</p>
            <button onclick="showPage('appDashboard')" class="w-full bg-indigo-600 text-white py-4 rounded-xl font-bold shadow-md hover:bg-indigo-700 transition">Enter Console</button>
        </div>
    </div>

    <div id="appDashboard" class="hidden min-h-screen">
        <nav class="bg-indigo-600 text-white p-4">
            <div class="max-w-6xl mx-auto flex justify-between items-center">
                <span class="font-bold">🏠 HomeschoolHero Console</span>
                <div class="flex gap-2">
                    <button onclick="exportToCSV()" class="text-xs bg-green-500 hover:bg-green-600 px-3 py-1 rounded">Download Excel/CSV</button>
                    <button onclick="clearAllData()" class="text-xs bg-red-500 hover:bg-red-600 px-3 py-1 rounded">Reset</button>
                </div>
            </div>
        </nav>
        
        <main class="max-w-5xl mx-auto p-8">
            <div class="bg-white p-8 rounded-3xl shadow-sm border mb-8">
                <h2 class="text-xl font-bold mb-4">What did we do today?</h2>
                <textarea id="userInput" class="w-full h-32 p-4 border rounded-2xl mb-4 text-lg outline-none focus:ring-2 focus:ring-indigo-500" placeholder="e.g. Read a book for 20 mins and self-corrected spelling..."></textarea>
                <button onclick="processActivity()" class="bg-indigo-600 text-white px-8 py-3 rounded-xl font-bold shadow-lg hover:shadow-indigo-200 transition">Translate & Save Activity</button>
            </div>

            <div id="subjectGrid" class="grid grid-cols-2 md:grid-cols-5 gap-4 mb-10">
                </div>

            <div class="bg-white p-6 rounded-3xl border">
                <h3 class="font-bold mb-4">Recent Translations</h3>
                <div id="logHistory" class="space-y-3 text-sm">
                    </div>
            </div>
        </main>
    </div>

    <script>
        const defaultSubjects = {
            ela: { name: "Language Arts", color: "bg-blue-500", hours: 0, target: 120, keywords: ["read", "write", "spelled", "grammar", "editing", "book"] },
            math: { name: "Mathematics", color: "bg-green-500", hours: 0, target: 120, keywords: ["measured", "calculated", "budget", "ratio", "fractions"] },
            science: { name: "Science", color: "bg-yellow-500", hours: 0, target: 120, keywords: ["yeast", "reaction", "nature", "experiment", "biology"] },
            history: { name: "Social Studies", color: "bg-red-500", hours: 0, target: 120, keywords: ["past", "map", "culture", "government", "history"] },
            fin: { name: "Financial Lit", color: "bg-emerald-500", hours: 0, target: 60, keywords: ["cost", "tax", "price", "savings", "interest"] },
            ent: { name: "Entrepreneur", color: "bg-purple-500", hours: 0, target: 60, keywords: ["product", "marketing", "business", "brand", "sales"] },
            lang: { name: "World Language", color: "bg-pink-500", hours: 0, target: 120, keywords: ["spanish", "translation", "phrases", "video"] },
            arts: { name: "The Arts", color: "bg-orange-500", hours: 0, target: 60, keywords: ["sketch", "design", "music", "color", "art"] },
            pe: { name: "Physical Ed", color: "bg-cyan-500", hours: 0, target: 60, keywords: ["exercise", "hiking", "sport", "cleaning", "yard"] },
            tech: { name: "Technology", color: "bg-slate-700", hours: 0, target: 60, keywords: ["gaming", "coding", "software", "youtube", "digital"] }
        };

        let userProgress = JSON.parse(localStorage.getItem('hsHeroProgress')) || defaultSubjects;
        let historyLogs = JSON.parse(localStorage.getItem('hsHeroLogs')) || [];

        function showPage(pageId) {
            document.querySelectorAll('body > div').forEach(div => div.classList.add('hidden'));
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
                    <div class="bg-white p-4 rounded-2xl border border-slate-100 shadow-sm">
                        <h3 class="text-xs font-bold text-slate-500 uppercase mb-2">${sub.name}</h3>
                        <div class="text-lg font-bold mb-2">${sub.hours.toFixed(1)}h</div>
                        <div class="w-full bg-slate-100 h-2 rounded-full overflow-hidden">
                            <div class="progress-bar ${sub.color} h-full" style="width: ${perc}%"></div>
                        </div>
                    </div>`;
            }

            const historyDiv = document.getElementById('logHistory');
            historyDiv.innerHTML = historyLogs.map(log => `
                <div class="p-3 bg-slate-50 rounded-lg border-l-4 border-indigo-400">
                    <span class="text-[10px] font-bold text-slate-400">${log.date}</span>
                    <p class="italic text-slate-700">"${log.text}"</p>
                </div>
            `).join('') || '<p class="text-slate-400">No logs yet.</p>';
        }

        function processActivity() {
            const input = document.getElementById('userInput');
            const text = input.value.toLowerCase();
            if (!text) return;

            let foundMatch = false;
            for (let key in userProgress) {
                userProgress[key].keywords.forEach(word => {
                    if (text.includes(word)) {
                        userProgress[key].hours += 0.5;
                        foundMatch = true;
                    }
                });
            }

            if(foundMatch) {
                historyLogs.unshift({ date: new Date().toLocaleString(), text: input.value });
                saveData();
                renderApp();
                input.value = '';
            } else {
                alert("Try being more descriptive to earn credits!");
            }
        }

        function saveData() {
            localStorage.setItem('hsHeroProgress', JSON.stringify(userProgress));
            localStorage.setItem('hsHeroLogs', JSON.stringify(historyLogs));
        }

        function exportToCSV() {
            let csvContent = "data:text/csv;charset=utf-8,Date,Activity Log\n";
            historyLogs.forEach(log => {
                csvContent += `"${log.date}","${log.text.replace(/"/g, '""')}"\n`;
            });
            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", "homeschool_transcript.csv");
            document.body.appendChild(link);
            link.click();
        }

        function clearAllData() {
            if(confirm("Wipe all data?")) {
                localStorage.clear();
                location.reload();
            }
        }
    </script>
</body>
</html>
