<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="robots" content="noindex, nofollow">
  <title>Work Hub | Хмарна панель</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>
    body { font-family: 'Plus Jakarta Sans', sans-serif; }
  </style>

  <!-- Підключення Firebase SDK -->
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
    import { getFirestore, doc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";

    // Твої ключі з консолі Firebase
    const firebaseConfig = {
      apiKey: "AizaSyA7w1TR4TE1ziaBG0dKzI92nxuoGEM1lqw",
      authDomain: "work-vau.firebaseapp.com",
      projectId: "work-vau",
      storageBucket: "work-vau.firebasestorage.app",
      messagingSenderId: "688337728430",
      appId: "1:688337728430:web:436f588e1aef4bf3474e3f"
    };

    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);
    const docRef = doc(db, "workspace", "main_dashboard");

    let state = {
      links: [],
      notes: []
    };

    // Синхронізація у реальному часі між ПК вдома та на роботі
    onSnapshot(docRef, (docSnap) => {
      if (docSnap.exists()) {
        state = docSnap.data();
      } else {
        state = {
          links: [
            { id: 1, title: 'Контент-план (Таблиця)', category: 'Робота', type: 'sheet', url: 'https://sheets.google.com' },
            { id: 2, title: 'Матеріали / Фотосесії', category: 'Google Диск', type: 'drive', url: 'https://drive.google.com' }
          ],
          notes: [
            { id: 1, title: 'Ідеї для контенту 💡', content: '• Розбір помилок у візуалі\n• Бекстейдж роботи над проєктом' }
          ]
        };
        saveToCloud();
      }
      renderLinks();
      renderNotes();
      updateSyncStatus('Синхронізовано з хмарою');
    }, (error) => {
      console.error(error);
      updateSyncStatus('Помилка доступу до бази', true);
    });

    async function saveToCloud() {
      updateSyncStatus('Збереження у хмару...');
      try {
        await setDoc(docRef, state);
        updateSyncStatus('Синхронізовано з хмарою');
      } catch (e) {
        console.error("Помилка збереження: ", e);
        updateSyncStatus('Помилка збереження', true);
      }
    }

    function updateSyncStatus(text, isError = false) {
      const statusEl = document.getElementById('sync-status');
      const dot = document.getElementById('sync-dot');
      if (!statusEl || !dot) return;
      statusEl.textContent = text;
      if (isError) {
        dot.className = 'w-2 h-2 rounded-full bg-rose-500 animate-pulse';
        statusEl.className = 'text-rose-400 text-xs font-medium';
      } else {
        dot.className = 'w-2 h-2 rounded-full bg-emerald-400 animate-pulse';
        statusEl.className = 'text-emerald-400 text-xs font-medium';
      }
    }

    // Рендер посилань
    window.renderLinks = function() {
      const grid = document.getElementById('links-grid');
      grid.innerHTML = '';
      (state.links || []).forEach(item => {
        let icon = '🔗';
        if (item.type === 'sheet') icon = '📊';
        else if (item.type === 'drive') icon = '📁';
        else if (item.type === 'doc') icon = '📄';

        const card = document.createElement('div');
        card.className = 'group relative bg-slate-900/90 hover:bg-slate-800/90 border border-slate-800 p-4 rounded-xl flex items-center justify-between shadow-sm transition duration-150';
        card.innerHTML = `
          <div class="flex items-center gap-3">
            <span class="text-xl p-2 rounded-lg bg-slate-950 border border-slate-800">${icon}</span>
            <div>
              <a href="${item.url}" target="_blank" rel="noopener noreferrer" class="font-medium text-sm text-slate-100 hover:text-indigo-400 transition flex items-center gap-1">
                ${item.title} <span class="text-[10px] text-slate-500">↗</span>
              </a>
              <span class="text-[11px] text-slate-400">${item.category}</span>
            </div>
          </div>
          <button onclick="deleteLink(${item.id})" class="text-slate-600 hover:text-rose-400 text-xs p-1 opacity-0 group-hover:opacity-100 transition" title="Видалити">✕</button>
        `;
        grid.appendChild(card);
      });
    };

    // Рендер нотаток
    window.renderNotes = function() {
      const grid = document.getElementById('notes-grid');
      grid.innerHTML = '';
      (state.notes || []).forEach(note => {
        const el = document.createElement('div');
        el.className = 'bg-slate-900/90 border border-slate-800 rounded-xl p-4 shadow-sm flex flex-col justify-between space-y-2 group';
        el.innerHTML = `
          <div>
            <div class="flex items-center justify-between gap-2 border-b border-slate-800/80 pb-2">
              <input type="text" value="${note.title}" onchange="updateNoteTitle(${note.id}, this.value)" class="bg-transparent text-sm font-semibold text-white focus:outline-none focus:text-indigo-400 w-full" placeholder="Заголовок...">
              <button onclick="deleteNote(${note.id})" class="text-slate-600 hover:text-rose-400 text-xs opacity-0 group-hover:opacity-100 transition" title="Видалити">✕</button>
            </div>
            <textarea oninput="updateNoteContent(${note.id}, this.value)" rows="5" class="w-full bg-transparent text-xs text-slate-300 focus:outline-none resize-none pt-2 font-mono leading-relaxed" placeholder="Введіть текст нотатки...">${note.content}</textarea>
          </div>
          <span class="text-[10px] text-slate-600 text-right">Хмарне збереження</span>
        `;
        grid.appendChild(el);
      });
    };

    window.saveNewLink = function() {
      const title = document.getElementById('modal-title').value.trim();
      const category = document.getElementById('modal-category').value.trim() || 'Загальне';
      const type = document.getElementById('modal-type').value;
      const url = document.getElementById('modal-url').value.trim();

      if (!title || !url) return alert('Будь ласка, заповніть назву та посилання');

      state.links.push({ id: Date.now(), title, category, type, url });
      saveToCloud();
      closeModal();
    };

    window.deleteLink = function(id) {
      state.links = state.links.filter(i => i.id !== id);
      saveToCloud();
    };

    window.addNote = function() {
      state.notes.unshift({ id: Date.now(), title: 'Нова замітка', content: '' });
      saveToCloud();
    };

    window.updateNoteTitle = function(id, val) {
      const n = state.notes.find(i => i.id === id);
      if (n) { n.title = val; saveToCloud(); }
    };

    let debounceTimer;
    window.updateNoteContent = function(id, val) {
      const n = state.notes.find(i => i.id === id);
      if (n) {
        n.content = val;
        clearTimeout(debounceTimer);
        debounceTimer = setTimeout(() => { saveToCloud(); }, 500);
      }
    };

    window.deleteNote = function(id) {
      state.notes = state.notes.filter(i => i.id !== id);
      saveToCloud();
    };
  </script>
</head>
<body class="bg-[#0b0f19] text-slate-100 min-h-screen pb-16">

  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pt-8 space-y-8">
    
    <!-- Шапка -->
    <header class="flex flex-col md:flex-row md:items-center justify-between gap-4 pb-6 border-b border-slate-800/80">
      <div class="flex items-center gap-3">
        <span class="p-2.5 bg-indigo-500/10 text-indigo-400 border border-indigo-500/20 rounded-xl text-xl">⚡</span>
        <div>
          <h1 class="text-2xl sm:text-3xl font-bold tracking-tight text-white">Work Hub</h1>
          <p id="current-date" class="text-xs sm:text-sm text-slate-400 capitalize mt-0.5"></p>
        </div>
      </div>
      
      <!-- Статус хмари -->
      <div class="flex items-center gap-2.5 px-3.5 py-1.5 rounded-full bg-slate-900 border border-slate-800 shadow-sm w-fit">
        <span id="sync-dot" class="w-2 h-2 rounded-full bg-amber-400 animate-pulse"></span>
        <span id="sync-status" class="text-xs text-slate-400">Підключення до Firebase...</span>
      </div>
    </header>

    <!-- Верхня сітка: Посилання та Калькулятор -->
    <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">
      
      <!-- Розділ посилань -->
      <section class="lg:col-span-7 space-y-4">
        <div class="flex items-center justify-between">
          <h2 class="text-lg font-semibold text-slate-200 flex items-center gap-2">
            <span class="text-indigo-400">📁</span> Робочі таблиці та диски
          </h2>
          <button onclick="openModal()" class="text-xs bg-indigo-600 hover:bg-indigo-500 active:scale-95 text-white font-medium px-3.5 py-1.5 rounded-lg transition shadow-sm">
            + Додати посилання
          </button>
        </div>

        <div id="links-grid" class="grid grid-cols-1 sm:grid-cols-2 gap-3.5">
          <!-- Генерується динамічно -->
        </div>
      </section>

      <!-- SMM Калькулятор -->
      <section class="lg:col-span-5 space-y-4">
        <h2 class="text-lg font-semibold text-slate-200 flex items-center gap-2">
          <span class="text-emerald-400">📊</span> SMM Калькулятор
        </h2>

        <div class="bg-slate-900/90 border border-slate-800 rounded-2xl p-5 shadow-lg space-y-5">
          <div class="flex p-1 bg-slate-950/80 rounded-xl border border-slate-800 text-xs font-medium">
            <button id="tab-err-btn" onclick="switchCalcTab('err')" class="flex-1 py-1.5 rounded-lg bg-indigo-600 text-white transition">ERR Поста</button>
            <button id="tab-stories-btn" onclick="switchCalcTab('stories')" class="flex-1 py-1.5 rounded-lg text-slate-400 hover:text-white transition">Stories</button>
          </div>

          <!-- Блок ERR -->
          <div id="calc-err" class="space-y-3">
            <div>
              <label class="block text-[11px] font-medium text-slate-400 uppercase tracking-wider mb-1">Охоплення публікації (Reach)</label>
              <input type="number" id="err-reach" placeholder="10000" class="w-full bg-slate-950 border border-slate-700/70 rounded-xl px-3.5 py-2 text-sm text-white focus:outline-none focus:border-indigo-500">
            </div>

            <div class="grid grid-cols-2 gap-2.5">
              <div><label class="block text-[11px] text-slate-400 mb-1">Лайки</label><input type="number" id="err-likes" placeholder="450" class="w-full bg-slate-950 border border-slate-700/70 rounded-lg px-3 py-1.5 text-sm text-white"></div>
              <div><label class="block text-[11px] text-slate-400 mb-1">Коментарі</label><input type="number" id="err-comments" placeholder="32" class="w-full bg-slate-950 border border-slate-700/70 rounded-lg px-3 py-1.5 text-sm text-white"></div>
              <div><label class="block text-[11px] text-slate-400 mb-1">Збереження</label><input type="number" id="err-saves" placeholder="85" class="w-full bg-slate-950 border border-slate-700/70 rounded-lg px-3 py-1.5 text-sm text-white"></div>
              <div><label class="block text-[11px] text-slate-400 mb-1">Репости</label><input type="number" id="err-shares" placeholder="40" class="w-full bg-slate-950 border border-slate-700/70 rounded-lg px-3 py-1.5 text-sm text-white"></div>
            </div>

            <div class="mt-4 p-3 bg-slate-950/60 rounded-xl border border-slate-800 flex items-center justify-between">
              <div>
                <span class="text-[11px] text-slate-400 block">Результат ERR</span>
                <span id="err-output" class="text-2xl font-bold text-emerald-400">0.00%</span>
              </div>
              <span id="err-verdict" class="text-xs px-2.5 py-1 rounded-md bg-slate-800 text-slate-400">Очікування даних</span>
            </div>
          </div>

          <!-- Блок Stories -->
          <div id="calc-stories" class="space-y-3 hidden">
            <div>
              <label class="block text-[11px] font-medium text-slate-400 uppercase tracking-wider mb-1">Підписники акаунту</label>
              <input type="number" id="st-followers" placeholder="5000" class="w-full bg-slate-950 border border-slate-700/70 rounded-xl px-3.5 py-2 text-sm text-white">
            </div>
            <div>
              <label class="block text-[11px] font-medium text-slate-400 uppercase tracking-wider mb-1">Перегляди сторіз через кому</label>
              <input type="text" id="st-views" placeholder="1000, 950, 880, 810" class="w-full bg-slate-950 border border-slate-700/70 rounded-xl px-3.5 py-2 text-sm text-white">
            </div>

            <div class="grid grid-cols-2 gap-2 mt-4">
              <div class="p-3 bg-slate-950/60 rounded-xl border border-slate-800">
                <span class="text-[11px] text-slate-400 block">Сер. перегляди</span>
                <span id="st-avg-output" class="text-lg font-bold text-sky-400">0</span>
                <span id="st-reach-rate" class="text-[10px] text-slate-400 block mt-0.5">0% від ауд.</span>
              </div>
              <div class="p-3 bg-slate-950/60 rounded-xl border border-slate-800">
                <span class="text-[11px] text-slate-400 block">Додивляння</span>
                <span id="st-retention-output" class="text-lg font-bold text-sky-400">0.0%</span>
                <span class="text-[10px] text-slate-500 block mt-0.5">перша → остання</span>
              </div>
            </div>
          </div>

          <button onclick="copyMetricsSummary()" class="w-full py-2 bg-slate-800 hover:bg-slate-700 text-slate-200 text-xs font-semibold rounded-xl transition flex items-center justify-center gap-2 border border-slate-700/60">
            <span>📋</span> Скопіювати звіт
          </button>
        </div>
      </section>
    </div>

    <!-- Розділ Нотаток -->
    <section class="space-y-4 pt-4 border-t border-slate-800/80">
      <div class="flex items-center justify-between">
        <div>
          <h2 class="text-lg font-semibold text-slate-200 flex items-center gap-2">
            <span class="text-amber-400">📝</span> Робочі нотатки
          </h2>
          <p class="text-xs text-slate-400">Миттєва хмарна синхронізація з будь-якого комп'ютера</p>
        </div>
        <button onclick="addNote()" class="text-xs bg-amber-500/10 hover:bg-amber-500/20 text-amber-300 border border-amber-500/20 font-medium px-3.5 py-1.5 rounded-lg transition">
          + Нова нотатка
        </button>
      </div>

      <div id="notes-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        <!-- Генерується динамічно -->
      </div>
    </section>

  </div>

  <!-- Модальне вікно для посилань -->
  <div id="link-modal" class="fixed inset-0 bg-black/70 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
    <div class="bg-slate-900 border border-slate-800 rounded-2xl max-w-md w-full p-6 space-y-4 shadow-2xl">
      <div class="flex items-center justify-between border-b border-slate-800 pb-3">
        <h3 class="font-semibold text-white">Додати посилання</h3>
        <button onclick="closeModal()" class="text-slate-400 hover:text-white">✕</button>
      </div>
      <div class="space-y-3 text-sm">
        <div>
          <label class="block text-xs text-slate-400 mb-1">Назва картки</label>
          <input type="text" id="modal-title" placeholder="Звіти Вересень" class="w-full bg-slate-950 border border-slate-700 rounded-xl px-3 py-2 text-white">
        </div>
        <div>
          <label class="block text-xs text-slate-400 mb-1">Категорія / Клієнт</label>
          <input type="text" id="modal-category" placeholder="Основний проєкт" class="w-full bg-slate-950 border border-slate-700 rounded-xl px-3 py-2 text-white">
        </div>
        <div>
          <label class="block text-xs text-slate-400 mb-1">Тип</label>
          <select id="modal-type" class="w-full bg-slate-950 border border-slate-700 rounded-xl px-3 py-2 text-white">
            <option value="sheet">📊 Google Таблиця</option>
            <option value="drive">📁 Google Диск</option>
            <option value="doc">📄 Google Документ</option>
            <option value="other">🔗 Інше посилання</option>
          </select>
        </div>
        <div>
          <label class="block text-xs text-slate-400 mb-1">URL</label>
          <input type="url" id="modal-url" placeholder="https://docs.google.com/..." class="w-full bg-slate-950 border border-slate-700 rounded-xl px-3 py-2 text-white">
        </div>
      </div>
      <div class="flex justify-end gap-2 pt-2">
        <button onclick="closeModal()" class="px-4 py-2 text-xs rounded-xl text-slate-400 hover:text-white">Скасувати</button>
        <button onclick="saveNewLink()" class="px-4 py-2 text-xs bg-indigo-600 hover:bg-indigo-500 text-white rounded-xl">Зберегти</button>
      </div>
    </div>
  </div>

  <script>
    document.getElementById('current-date').textContent = new Intl.DateTimeFormat('uk-UA', { weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' }).format(new Date());

    function openModal() { document.getElementById('link-modal').classList.remove('hidden'); }
    function closeModal() {
      document.getElementById('link-modal').classList.add('hidden');
      document.getElementById('modal-title').value = '';
      document.getElementById('modal-category').value = '';
      document.getElementById('modal-url').value = '';
    }

    // Перемикання вкладок калькулятора
    function switchCalcTab(tab) {
      document.getElementById('calc-err').classList.toggle('hidden', tab !== 'err');
      document.getElementById('calc-stories').classList.toggle('hidden', tab !== 'stories');
      document.getElementById('tab-err-btn').className = tab === 'err' ? 'flex-1 py-1.5 rounded-lg bg-indigo-600 text-white transition' : 'flex-1 py-1.5 rounded-lg text-slate-400 hover:text-white transition';
      document.getElementById('tab-stories-btn').className = tab === 'stories' ? 'flex-1 py-1.5 rounded-lg bg-indigo-600 text-white transition' : 'flex-1 py-1.5 rounded-lg text-slate-400 hover:text-white transition';
    }

    ['err-reach', 'err-likes', 'err-comments', 'err-saves', 'err-shares', 'st-followers', 'st-views'].forEach(id => {
      document.getElementById(id).addEventListener('input', runCalculations);
    });

    function runCalculations() {
      const reach = parseFloat(document.getElementById('err-reach').value) || 0;
      const likes = parseFloat(document.getElementById('err-likes').value) || 0;
      const comments = parseFloat(document.getElementById('err-comments').value) || 0;
      const saves = parseFloat(document.getElementById('err-saves').value) || 0;
      const shares = parseFloat(document.getElementById('err-shares').value) || 0;

      let err = reach > 0 ? ((likes + comments + saves + shares) / reach) * 100 : 0;
      document.getElementById('err-output').textContent = err.toFixed(2) + '%';

      const verdict = document.getElementById('err-verdict');
      if (reach === 0) verdict.textContent = 'Очікування даних';
      else if (err >= 8) verdict.textContent = '🔥 Топ результат';
      else if (err >= 4) verdict.textContent = '👍 Хороша норма';
      else verdict.textContent = '📉 Низький ER';

      const followers = parseFloat(document.getElementById('st-followers').value) || 0;
      const rawStories = document.getElementById('st-views').value;
      const views = rawStories.split(',').map(n => parseFloat(n.trim())).filter(n => !isNaN(n) && n > 0);

      if (views.length > 0) {
        const avg = Math.round(views.reduce((a, b) => a + b, 0) / views.length);
        document.getElementById('st-avg-output').textContent = avg.toLocaleString();
        document.getElementById('st-reach-rate').textContent = followers > 0 ? `${((avg / followers) * 100).toFixed(1)}% від ауд.` : 'введіть підписників';
        document.getElementById('st-retention-output').textContent = `${((views[views.length - 1] / views[0]) * 100).toFixed(1)}%`;
      } else {
        document.getElementById('st-avg-output').textContent = '0';
        document.getElementById('st-retention-output').textContent = '0.0%';
      }
    }

    function copyMetricsSummary() {
      const err = document.getElementById('err-output').textContent;
      const avg = document.getElementById('st-avg-output').textContent;
      const rate = document.getElementById('st-reach-rate').textContent;
      const ret = document.getElementById('st-retention-output').textContent;
      navigator.clipboard.writeText(`📊 SMM Звіт:\n• ERR: ${err}\n• Stories сер.: ${avg} (${rate})\n• Додивляння: ${ret}`).then(() => alert('Звіт скопійовано!'));
    }
  </script>
</body>
</html>
