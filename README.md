<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rotinas Mágicas da Alice</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Quicksand:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Quicksand', sans-serif;
            background-color: #fdfcf0;
            background-image: radial-gradient(#e0f2fe 1px, transparent 1px);
            background-size: 30px 30px;
        }
        .card-task {
            transition: all 0.2s ease-out;
            cursor: pointer;
            user-select: none;
        }
        .card-task:active { transform: scale(0.95); }
        .task-done {
            background-color: #dcfce7 !important;
            border-color: #4ade80 !important;
        }
        .day-active {
            background-color: #0ea5e9 !important;
            color: white !important;
            box-shadow: 0 4px 10px rgba(14, 165, 233, 0.4);
        }
        .animate-pop { animation: pop 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        @keyframes pop {
            0% { transform: scale(0.8); opacity: 0; }
            100% { transform: scale(1); opacity: 1; }
        }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body class="pb-32">

    <div class="max-w-4xl mx-auto px-4 py-8">
        <!-- Cabeçalho -->
        <header class="text-center mb-8">
            <h1 class="text-4xl font-bold text-sky-600 mb-2">Rotinas da Alice ✨</h1>
            <div id="date-display" class="text-sky-400 font-bold text-lg uppercase tracking-widest"></div>
        </header>

        <!-- Seletor de Dias -->
        <div class="flex overflow-x-auto gap-2 pb-4 mb-8 no-scrollbar justify-center">
            <button onclick="setDay('Segunda')" class="day-btn px-4 py-2 rounded-full bg-white border-2 border-sky-100 font-bold text-sky-600 transition-all">Seg</button>
            <button onclick="setDay('Terça')" class="day-btn px-4 py-2 rounded-full bg-white border-2 border-sky-100 font-bold text-sky-600 transition-all">Ter</button>
            <button onclick="setDay('Quarta')" class="day-btn px-4 py-2 rounded-full bg-white border-2 border-sky-100 font-bold text-sky-600 transition-all">Qua</button>
            <button onclick="setDay('Quinta')" class="day-btn px-4 py-2 rounded-full bg-white border-2 border-sky-100 font-bold text-sky-600 transition-all">Qui</button>
            <button onclick="setDay('Sexta')" class="day-btn px-4 py-2 rounded-full bg-white border-2 border-sky-100 font-bold text-sky-600 transition-all">Sex</button>
            <button onclick="setDay('Sábado')" class="day-btn px-4 py-2 rounded-full bg-white border-2 border-orange-100 font-bold text-orange-500 transition-all">Sáb</button>
            <button onclick="setDay('Domingo')" class="day-btn px-4 py-2 rounded-full bg-white border-2 border-orange-100 font-bold text-orange-500 transition-all">Dom</button>
        </div>

        <!-- Conteúdo Principal -->
        <div id="planner-grid" class="space-y-10"></div>

        <!-- Botões de Reset -->
        <div class="mt-12 mb-20 flex flex-col items-center gap-4">
            <button onclick="resetCurrentDay()" class="text-xs text-gray-400 hover:text-red-400 underline transition-colors">Recomeçar este dia</button>
            <button onclick="resetFullWeek()" class="text-[10px] text-gray-300 hover:text-red-500 uppercase tracking-tighter transition-colors font-bold">Zerar a semana toda</button>
        </div>

        <!-- Barra de Progresso Inferior -->
        <div class="fixed bottom-6 left-1/2 -translate-x-1/2 w-[90%] max-w-lg bg-white/90 backdrop-blur-md border-2 border-sky-200 p-4 rounded-[2rem] shadow-2xl flex items-center justify-between z-50">
            <div class="flex-1 mr-4">
                <p id="progress-label" class="text-sky-600 font-bold text-xs mb-1 uppercase tracking-tight">PROGRESSO</p>
                <div class="w-full h-4 bg-gray-100 rounded-full overflow-hidden border border-gray-100">
                    <div id="progress-bar" class="h-full bg-gradient-to-r from-yellow-400 to-green-500 transition-all duration-700" style="width: 0%"></div>
                </div>
            </div>
            <div id="percent-text" class="text-2xl font-black text-sky-600">0%</div>
        </div>
    </div>

    <script>
        const config = [
            { titulo: "Manhã", icone: "☀️", cor: "text-yellow-600", tarefas: ["Trocar de roupa", "Tomar o lanche", "Escovar os dentes", "Pentear o cabelo", "Ir para a escola"] },
            { titulo: "Tarde", icone: "🌤️", cor: "text-orange-600", tarefas: ["Tomar banho", "Hora do brinquedo", "Hora do jantar", "Hora livre"] },
            { titulo: "Noite", icone: "🌙", cor: "text-indigo-600", tarefas: ["Escovar os dentes", "Colocar o pijama", "Ler uma história", "Fazer oração"] },
            { titulo: "Bônus", icone: "🏆", cor: "text-pink-600", tarefas: ["Cuidar dos brinquedos", "Não gritar ou brigar", "Obedecer papai e mamãe"] }
        ];

        let diaAtual = 'Segunda';
        let dados = JSON.parse(localStorage.getItem('planner_alice_v2')) || {
            'Segunda': {}, 'Terça': {}, 'Quarta': {}, 'Quinta': {}, 'Sexta': {}, 'Sábado': {}, 'Domingo': {}
        };

        function setDay(dia) {
            diaAtual = dia;
            document.querySelectorAll('.day-btn').forEach(b => {
                b.classList.remove('day-active');
                if(b.innerText.startsWith(dia.substring(0,3))) b.classList.add('day-active');
            });
            document.getElementById('date-display').innerText = (dia === 'Sábado' || dia === 'Domingo') ? dia : dia + '-Feira';
            document.getElementById('progress-label').innerText = `Progresso de ${dia}`;
            render();
        }

        function toggleTask(tarefa) {
            dados[diaAtual][tarefa] = !dados[diaAtual][tarefa];
            localStorage.setItem('planner_alice_v2', JSON.stringify(dados));
            if(dados[diaAtual][tarefa]) {
                confetti({ particleCount: 40, spread: 60, origin: { y: 0.8 }, colors: ['#4ade80', '#38bdf8'] });
            }
            render();
        }

        function resetCurrentDay() {
            if(confirm("Queres recomeçar o dia de " + diaAtual + "?")) {
                dados[diaAtual] = {};
                localStorage.setItem('planner_alice_v2', JSON.stringify(dados));
                render();
            }
        }

        function resetFullWeek() {
            if(confirm("ATENÇÃO: Queres apagar o progresso de TODOS os dias da semana?")) {
                dados = { 'Segunda': {}, 'Terça': {}, 'Quarta': {}, 'Quinta': {}, 'Sexta': {}, 'Sábado': {}, 'Domingo': {} };
                localStorage.setItem('planner_alice_v2', JSON.stringify(dados));
                render();
            }
        }

        function render() {
            const grid = document.getElementById('planner-grid');
            grid.innerHTML = '';
            let total = 0, feitos = 0;

            config.forEach(sec => {
                const sectionHtml = `
                    <div class="animate-pop">
                        <div class="flex items-center gap-2 mb-4">
                            <span class="text-2xl">${sec.icone}</span>
                            <h2 class="text-xl font-bold ${sec.cor}">${sec.titulo}</h2>
                        </div>
                        <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                            ${sec.tarefas.map(t => {
                                total++;
                                const isDone = dados[diaAtual][t];
                                if(isDone) feitos++;
                                return `
                                    <div onclick="toggleTask('${t}')" class="card-task p-4 rounded-2xl border-2 flex items-center justify-between bg-white ${isDone ? 'task-done' : 'border-sky-50 shadow-sm'}">
                                        <div class="flex items-center gap-3">
                                            <span class="text-lg">${getEmoji(t)}</span>
                                            <span class="font-semibold text-gray-700">${t}</span>
                                        </div>
                                        <div class="w-6 h-6 rounded-full border-2 flex items-center justify-center ${isDone ? 'bg-green-500 border-green-500' : 'border-gray-200'}">
                                            ${isDone ? '<svg class="w-4 h-4 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"></path></svg>' : ''}
                                        </div>
                                    </div>
                                `;
                            }).join('')}
                        </div>
                    </div>
                `;
                grid.innerHTML += sectionHtml;
            });

            const pct = Math.round((feitos/total)*100);
            document.getElementById('progress-bar').style.width = pct + '%';
            document.getElementById('percent-text').innerText = pct + '%';
            if(pct === 100) confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } });
        }

        function getEmoji(t) {
            const map = { "Trocar de roupa": "👕", "Tomar o lanche": "🍎", "Escovar os dentes": "🪥", "Pentear o cabelo": "🪮", "Ir para a escola": "🎒", "Tomar banho": "🛀", "Hora do brinquedo": "🧸", "Hora do jantar": "🍝", "Hora livre": "🎨", "Colocar o pijama": "💤", "Ler uma história": "📚", "Fazer oração": "🙏", "Cuidar dos brinquedos": "🧹", "Não gritar ou brigar": "🤫", "Obedecer papai e mamãe": "❤️" };
            return map[t] || "✨";
        }

        const diasSemana = ['Domingo', 'Segunda', 'Terça', 'Quarta', 'Quinta', 'Sexta', 'Sábado'];
        setDay(diasSemana[new Date().getDay()]);
    </script>
</body>
</html># planner-alice
