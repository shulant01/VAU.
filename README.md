<!DOCTYPE html>
<html lang="uk" class="h-full">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="robots" content="noindex, nofollow">
  <title>WORKSPACE</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
  <style>
    body {
      font-family: 'Plus Jakarta Sans', sans-serif;
      user-select: none;
    }
    input, textarea {
      user-select: text;
    }
    .font-mono-code {
      font-family: 'JetBrains Mono', monospace;
    }
    /* Видалення стрілочок з полів */
    input[type=number]::-webkit-inner-spin-button, 
    input[type=number]::-webkit-outer-spin-button { 
      -webkit-appearance: none; 
      margin: 0; 
    }
    input[type=number] {
      -moz-appearance: textfield;
    }
    /* Кастомний тонкий скролбар для списку карток за потреби */
    ::-webkit-scrollbar {
      width: 4px;
      height: 4px;
    }
    ::-webkit-scrollbar-track {
      background: transparent;
    }
    ::-webkit-scrollbar-thumb {
      background: #27272a;
      border-radius: 4px;
    }
  </style>

  <!-- Firebase Модуль -->
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
    import { getFirestore, doc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";

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
    const docRef = doc(db, "workspace", "compact_dashboard");

    let state = {
      links: [],
      notes: ""
    };

    // Слухач хмари в реальному часі
    onSnapshot(docRef, (docSnap) => {
      if (docSnap.exists()) {
        state = docSnap.data();
      } else {
        state = {
          links: [
            { id: 1, title: 'Таблиця звітів', category: 'Аналітика', type: 'sheet', url: 'https://sheets.google.com' },
            { id: 2, title: 'Робочий диск', category: 'Контент', type: 'drive', url: 'https://drive.google.com' },
            { id: 3, title: 'Сітка публікацій', category: 'План', type: 'sheet', url: 'https://sheets.google.com' }
          ],
          notes: "Швидкі нотатки, задачі, тези до зустрічей або копірайт..."
        };
        saveToCloud();
      }
      renderLinks();
      const notesEl = document.getElementById('notepad-area');
      if (document.activeElement !== notesEl) {
        notesEl.value = state.notes || "";
      }
      setSyncStatus(true);
    }, (error) => {
      console.error(error);
      setSyncStatus(false);
    });

    async function saveToCloud() {
      try {
        await setDoc(docRef, state);
        setSyncStatus(true);
      } catch (e) {
        console.error(e);
        setSyncStatus(false);
      }
    }

    function setSyncStatus(ok) {
      const dot = document.getElementById('sync-indicator');
      if (!dot) return;
      dot.className = ok ? 'w-2 h-2 rounded-full bg-zinc-400' : 'w-2 h-2 rounded-full bg-rose-500 animate-pulse';
    }

    // Рендер посилань
    window.renderLinks = function() {
      const container = document.getElementById('links-container');
      container.innerHTML = '';

      (state.links || []).forEach(item => {
        const card = document.createElement('div');
        card.className = 'group relative flex items-center justify-between p-3 rounded-xl bg-zinc-900/60 hover:bg-zinc-800/80 border border-zinc-800/70 hover:border-zinc-700 transition duration-150';
        
        let iconSvg = '';
        if (item.type === 'sheet') {
          iconSvg = `<svg class="w-4 h-4 text-zinc-400 group-hover:text-zinc-200" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.8"><path stroke-linecap="round" stroke-linejoin="round" d="M3 10h18M3 14h18m-9-4v8m-7 4h14a2 2 0 002-2V6a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"/></svg>`;
        } else if (item.type === 'drive') {
          iconSvg = `<svg class="w-4 h-4 text-zinc-400 group-hover:text-zinc-200" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.8"><path stroke-linecap="round" stroke-linejoin="round" d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-6l-2-2H5a2 2 0 00-2 2z"/></svg>`;
        } else {
          iconSvg = `<svg class="w-4 h-4 text-zinc-400 group-hover:text-zinc-200" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.8"><path stroke-linecap="round" stroke-linejoin="round" d="M13.828 10.172a4 4 0 00-5.656 0l-4 4a4 4 0 105.656 5.656l1.102-1.101m-.758-4.899a4 4 0 005.656 0l4-4a4 4 0 00-5.656-5.656l-1.1 1.1"/></svg>`;
        }

        card.innerHTML = `
          <div class="flex items-center gap-3 min-w-0 pr-2">
            <div class="w-8 h-8 rounded-lg bg-zinc-950 border border-zinc-800 flex items-center justify-center shrink-0">
              ${iconSvg}
            </div>
            <div class="min-w-0">
              <a href="${item.url}" target="_blank" rel="noopener noreferrer" class="text-xs font-semibold text-zinc-200 group-hover:text-white truncate block">
                ${item.title}
              </a>
              <span class="text-[10px] text-zinc-500 block truncate">${item.category || 'Посилання'}</span>
            </div>
          </div>
          <button onclick="deleteLink(${item.id})" class="opacity-0 group-hover:opacity-100 p-1 text-zinc-500 hover:text-zinc-300 transition" title="Видалити">
            <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/></svg>
          </button>
        `;
        container.appendChild(card);
      });
    };

    window.saveNewLink = function() {
      const title = document.getElementById('modal-title').value.trim();
      const category = document.getElementById('modal-cat').value.trim();
      const type = document.getElementById('modal-type').value;
      const url = document.getElementById('modal-url').value.trim();

      if (!title || !url) return;
      state.links.push({ id: Date.now(), title, category, type, url });
      saveToCloud();
      closeModal();
    };

    window.deleteLink = function(id) {
      state.links = state.links.filter(i => i.id !== id);
      saveToCloud();
    };

    // Збереження нотаток на льоту
    let noteTimer;
    window.onNoteInput = function(val) {
      state.notes = val;
      clearTimeout(noteTimer);
      noteTimer = setTimeout(() => {
        saveToCloud();
      }, 400);
    };
  </script>
</head>

<body class="h-screen w-screen bg-[#090a0f] text-zinc-300 antialiased overflow-hidden p-4 flex flex-col gap-3.5 selection:bg-zinc-700 selection:text-white">

  <!-- ВЕРХНІЙ БАР / HEADER -->
  <header class="w-full bg-[#11131a] border border-zinc-800/80 rounded-2xl px-5 py-3 flex items-center justify-between shrink-0 shadow-lg shadow-black/40">
    <div class="flex items-center gap-4">
      <h1 class="text-sm font-bold tracking-widest text-zinc-100 uppercase">WORKSPACE</h1>
      <span class="text-zinc-600 font-mono-code text-xs">/</span>
      <p id="cur-date" class="text-xs text-zinc-400 capitalize font-medium"></p>
    </div>

    <div class="flex items-center gap-3">
      <div class="flex items-center gap-2 px-3 py-1 rounded-full bg-zinc-950 border border-zinc-800 text-[11px] text-zinc-400 font-mono-code">
        <span id="sync-indicator" class="w-2 h-2 rounded-full bg-zinc-500"></span>
        <span>Cloud Sync</span>
      </div>
      <button onclick="openModal()" class="flex items-center gap-1.5 px-3 py-1.5 rounded-xl bg-zinc-100 hover:bg-white text-zinc-950 font-semibold text-xs transition duration-150 active:scale-95 shadow-sm">
        <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2.2"><path stroke-linecap="round" stroke-linejoin="round" d="M12 4v16m8-8H4"/></svg>
        <span>Новий ресурс</span>
      </button>
    </div>
  </header>

  <!-- ОСНОВНА СІТКА (1 СТОРІНКА БЕЗ СКРОЛУ) -->
  <main class="w-full flex-1 grid grid-cols-12 gap-3.5 min-h-0">
    
    <!-- КОЛОНКА 1: КАРТКИ ПОСИЛАНЬ ТА ТАБЛИЦЬ (3 колонки) -->
    <section class="col-span-3 bg-[#11131a] border border-zinc-800/80 rounded-2xl p-4 flex flex-col justify-between overflow-hidden shadow-lg shadow-black/30">
      <div class="flex items-center justify-between mb-3 shrink-0">
        <span class="text-xs font-semibold text-zinc-300 tracking-wide uppercase">Ресурси & Диски</span>
        <svg class="w-4 h-4 text-zinc-500" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.8"><path stroke-linecap="round" stroke-linejoin="round" d="M4 6h16M4 12h16M4 18h7"/></svg>
      </div>

      <!-- Список карток із внутрішнім компактним скролом за потреби -->
      <div id="links-container" class="flex-1 space-y-2 overflow-y-auto pr-1">
        <!-- Картки генеруються з Firebase -->
      </div>

      <div class="pt-3 border-t border-zinc-800/80 text-[10px] text-zinc-500 font-mono-code flex justify-between shrink-0">
        <span>Картки швидкого доступу</span>
        <span>1 клік ↗</span>
      </div>
    </section>

    <!-- КОЛОНКА 2: КАЛЬКУЛЯТОР ERR ТА СЕРЕДНІХ ПЕРЕГЛЯДІВ (4 колонки) -->
    <section class="col-span-4 bg-[#11131a] border border-zinc-800/80 rounded-2xl p-4 flex flex-col justify-between overflow-hidden shadow-lg shadow-black/30">
      
      <div class="flex items-center justify-between mb-3 shrink-0">
        <span class="text-xs font-semibold text-zinc-300 tracking-wide uppercase">Аналітика / Калькулятор</span>
        <svg class="w-4 h-4 text-zinc-500" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.8"><path stroke-linecap="round" stroke-linejoin="round" d="M9 7h6m0 10v-3m-3 3h.01M9 17h.01M9 14h.01M12 14h.01M15 11h.01M12 11h.01M9 11h.01M7 21h10a2 2 0 002-2V5a2 2 0 00-2-2H7a2 2 0 00-2 2v14a2 2 0 002 2z"/></svg>
      </div>

      <div class="flex-1 flex flex-col justify-between gap-3 min-h-0">

        <!-- БЛОК ERR -->
        <div class="bg-zinc-900/50 border border-zinc-800/70 rounded-xl p-3.5 flex flex-col justify-between">
          <div class="flex items-center justify-between mb-2">
            <span class="text-[11px] font-semibold text-zinc-300">ERR за охопленням</span>
            <div class="flex items-center gap-1.5">
              <span id="err-result" class="text-sm font-bold font-mono-code text-zinc-100">0.00%</span>
              <button onclick="copyValue('err-result')" class="p-1 rounded-md text-zinc-500 hover:text-zinc-200 hover:bg-zinc-800 transition" title="Скопіювати число">
                <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"/></svg>
              </button>
            </div>
          </div>

          <div class="space-y-2">
            <div>
              <label class="block text-[10px] text-zinc-500 font-mono-code mb-0.5">ОХОПЛЕННЯ (через пробіл)</label>
              <input type="text" id="err-reach-input" placeholder="500 233 43 847 294" class="w-full bg-zinc-950 border border-zinc-800 rounded-lg px-2.5 py-1.5 text-xs text-zinc-200 font-mono-code focus:outline-none focus:border-zinc-500 transition">
            </div>
            <div>
              <label class="block text-[10px] text-zinc-500 font-mono-code mb-0.5">ВЗАЄМОДІЇ (через пробіл)</label>
              <input type="text" id="err-eng-input" placeholder="33 978 23 42 83" class="w-full bg-zinc-950 border border-zinc-800 rounded-lg px-2.5 py-1.5 text-xs text-zinc-200 font-mono-code focus:outline-none focus:border-zinc-500 transition">
            </div>
          </div>
        </div>

        <!-- БЛОК СЕРЕДНІХ ПЕРЕГЛЯДІВ СТОРІЗ -->
        <div class="bg-zinc-900/50 border border-zinc-800/70 rounded-xl p-3.5 flex flex-col justify-between">
          <div class="flex items-center justify-between mb-2">
            <span class="text-[11px] font-semibold text-zinc-300">Середні Stories</span>
            <div class="flex items-center gap-1.5">
              <span id="stories-result" class="text-sm font-bold font-mono-code text-zinc-100">0</span>
              <button onclick="copyValue('stories-result')" class="p-1 rounded-md text-zinc-500 hover:text-zinc-200 hover:bg-zinc-800 transition" title="Скопіювати число">
                <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"/></svg>
              </button>
            </div>
          </div>

          <div>
            <label class="block text-[10px] text-zinc-500 font-mono-code mb-0.5">ПЕРЕГЛЯДИ (через пробіл)</label>
            <input type="text" id="stories-input" placeholder="200 200 450 120 310" class="w-full bg-zinc-950 border border-zinc-800 rounded-lg px-2.5 py-1.5 text-xs text-zinc-200 font-mono-code focus:outline-none focus:border-zinc-500 transition">
            <span class="text-[10px] text-zinc-600 block mt-1 font-mono-code">Розраховує середнє арифметичне</span>
          </div>
        </div>

      </div>

      <div class="pt-2 border-t border-zinc-800/80 text-[10px] text-zinc-500 font-mono-code flex justify-between shrink-0">
        <span>Миттєвий розрахунок</span>
        <span>Auto-calc</span>
      </div>
    </section>

    <!-- КОЛОНКА 3: ВЕЛИКЕ ПОЛЕ ДЛЯ НОТАТОК (SCRATCHPAD) (5 колонок) -->
    <section class="col-span-5 bg-[#11131a] border border-zinc-800/80 rounded-2xl p-4 flex flex-col justify-between overflow-hidden shadow-lg shadow-black/30">
      
      <div class="flex items-center justify-between mb-2 shrink-0">
        <div class="flex items-center gap-2">
          <span class="text-xs font-semibold text-zinc-300 tracking-wide uppercase">Швидкий Блокнот</span>
          <span class="text-[10px] text-zinc-500 font-mono-code">(Word-style pad)</span>
        </div>
        <svg class="w-4 h-4 text-zinc-500" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.8"><path stroke-linecap="round" stroke-linejoin="round" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"/></svg>
      </div>

      <!-- Повноцінний редактор без обмежень, прямо як Word / Notion -->
      <div class="flex-1 w-full min-h-0 bg-zinc-950/70 border border-zinc-800/80 rounded-xl p-3 flex flex-col">
        <textarea 
          id="notepad-area" 
          oninput="onNoteInput(this.value)" 
          placeholder="Почніть писати будь-який текст, чернетки постів, списки чи робочі замітки..." 
          class="w-full flex-1 bg-transparent text-xs text-zinc-200 focus:outline-none resize-none leading-relaxed font-sans"
        ></textarea>
      </div>

      <div class="pt-2 border-t border-zinc-800/80 text-[10px] text-zinc-500 font-mono-code flex justify-between shrink-0">
        <span>Зберігається у хмару автоматично</span>
        <span id="copy-note-feedback" class="text-zinc-400"></span>
      </div>
    </section>

  </main>

  <!-- МОДАЛЬНЕ ВІКНО ДОДАВАННЯ РЕСУРСУ -->
  <div id="modal-box" class="fixed inset-0 bg-black/80 backdrop-blur-xs z-50 hidden flex items-center justify-center p-4">
    <div class="bg-[#11131a] border border-zinc-800 rounded-2xl max-w-sm w-full p-5 space-y-3.5 shadow-2xl">
      <div class="flex items-center justify-between pb-2 border-b border-zinc-800">
        <span class="text-xs font-bold uppercase tracking-wider text-zinc-200">Додати ресурс</span>
        <button onclick="closeModal()" class="text-zinc-500 hover:text-zinc-200">
          <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/></svg>
        </button>
      </div>

      <div class="space-y-2.5 text-xs">
        <div>
          <label class="block text-[10px] text-zinc-500 font-mono-code mb-1">НАЗВА КАРТКИ</label>
          <input type="text" id="modal-title" placeholder="Таблиця звітів" class="w-full bg-zinc-950 border border-zinc-800 rounded-lg px-3 py-1.5 text-zinc-200 focus:outline-none focus:border-zinc-500">
        </div>
        <div>
          <label class="block text-[10px] text-zinc-500 font-mono-code mb-1">КАТЕГОРІЯ</label>
          <input type="text" id="modal-cat" placeholder="Аналітика" class="w-full bg-zinc-950 border border-zinc-800 rounded-lg px-3 py-1.5 text-zinc-200 focus:outline-none focus:border-zinc-500">
        </div>
        <div>
          <label class="block text-[10px] text-zinc-500 font-mono-code mb-1">ТИП</label>
          <select id="modal-type" class="w-full bg-zinc-950 border border-zinc-800 rounded-lg px-3 py-1.5 text-zinc-200 focus:outline-none focus:border-zinc-500">
            <option value="sheet">Таблиця (Sheets)</option>
            <option value="drive">Папка / Диск (Drive)</option>
            <option value="link">Посилання (Веб-сайт)</option>
          </select>
        </div>
        <div>
          <label class="block text-[10px] text-zinc-500 font-mono-code mb-1">URL</label>
          <input type="url" id="modal-url" placeholder="https://..." class="w-full bg-zinc-950 border border-zinc-800 rounded-lg px-3 py-1.5 text-zinc-200 focus:outline-none focus:border-zinc-500">
        </div>
      </div>

      <div class="flex justify-end gap-2 pt-2">
        <button onclick="closeModal()" class="px-3 py-1.5 text-xs text-zinc-400 hover:text-zinc-200">Скасувати</button>
        <button onclick="saveNewLink()" class="px-3 py-1.5 text-xs bg-zinc-100 hover:bg-white text-zinc-950 font-semibold rounded-lg">Зберегти</button>
      </div>
    </div>
  </div>

  <script>
    // Поточна дата
    const d = new Date();
    document.getElementById('cur-date').textContent = new Intl.DateTimeFormat('uk-UA', { weekday: 'short', day: 'numeric', month: 'short' }).format(d);

    // Модалка
    function openModal() { document.getElementById('modal-box').classList.remove('hidden'); }
    function closeModal() {
      document.getElementById('modal-box').classList.add('hidden');
      document.getElementById('modal-title').value = '';
      document.getElementById('modal-cat').value = '';
      document.getElementById('modal-url').value = '';
    }

    // --- РОЗУМНИЙ РОЗРАХУНОК ERR ТА STORIES (ПРИЙМАЄ ЧИСЛА БЕЗ ОБМЕЖЕНЬ ТА СИМВОЛІВ) ---
    function parseNumbers(str) {
      if (!str) return [];
      return str.match(/\d+(\.\d+)?/g)?.map(Number) || [];
    }

    function calculateMetrics() {
      // 1. ERR
      const reachArr = parseNumbers(document.getElementById('err-reach-input').value);
      const engArr = parseNumbers(document.getElementById('err-eng-input').value);

      const totalReach = reachArr.reduce((a, b) => a + b, 0);
      const totalEng = engArr.reduce((a, b) => a + b, 0);

      const errEl = document.getElementById('err-result');
      if (totalReach > 0) {
        const errVal = (totalEng / totalReach) * 100;
        errEl.textContent = errVal.toFixed(2) + '%';
      } else {
        errEl.textContent = '0.00%';
      }

      // 2. Stories Avg
      const storiesArr = parseNumbers(document.getElementById('stories-input').value);
      const storiesEl = document.getElementById('stories-result');

      if (storiesArr.length > 0) {
        const sum = storiesArr.reduce((a, b) => a + b, 0);
        const avg = Math.round(sum / storiesArr.length);
        storiesEl.textContent = avg.toLocaleString();
      } else {
        storiesEl.textContent = '0';
      }
    }

    document.getElementById('err-reach-input').addEventListener('input', calculateMetrics);
    document.getElementById('err-eng-input').addEventListener('input', calculateMetrics);
    document.getElementById('stories-input').addEventListener('input', calculateMetrics);

    // Копіювання самого результату
    function copyValue(elementId) {
      const text = document.getElementById(elementId).textContent;
      navigator.clipboard.writeText(text).then(() => {
        const original = document.getElementById(elementId).textContent;
        document.getElementById(elementId).textContent = 'Copied!';
        setTimeout(() => {
          document.getElementById(elementId).textContent = original;
        }, 800);
      });
    }
  </script>
</body>
</html>
