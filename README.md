<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Continuous List Calendar</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/lucide@latest"></script>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
  <!-- Official Google Identity Services SDK -->
  <script src="https://accounts.google.com/gsi/client" async defer></script>
  <style>
    body { font-family: 'Inter', sans-serif; }
    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-track { background: transparent; }
    ::-webkit-scrollbar-thumb { background: rgba(156, 163, 175, 0.4); border-radius: 9999px; }
    ::-webkit-scrollbar-thumb:hover { background: rgba(107, 114, 128, 0.6); }
    .day-highlight {
      box-shadow: 0 0 0 2px #d97706 !important;
      background-color: #fef3c7 !important;
      color: #92400e !important;
      font-weight: 600;
    }
  </style>
</head>
<body class="bg-zinc-100 text-zinc-900 h-screen w-screen flex flex-col overflow-hidden select-none">

  <!-- Header -->
  <header class="h-14 border-b border-zinc-200 bg-white flex items-center justify-between px-4 z-20 shrink-0">
    <div class="flex items-center space-x-3">
      <div class="flex items-center space-x-2">
        <i data-lucide="calendar" class="w-5 h-5 text-blue-600"></i>
        <span class="font-bold tracking-tight text-zinc-800 text-base hidden sm:inline">Continuous</span>
      </div>
      <button id="todayBtn" class="px-2.5 py-1 text-xs font-semibold rounded bg-zinc-100 hover:bg-zinc-200 border border-zinc-300 text-zinc-700 transition">
        Today
      </button>
      <input type="date" id="jumpDateInput" class="text-xs bg-zinc-50 border border-zinc-300 rounded px-2 py-1 text-zinc-700 focus:outline-none focus:ring-1 focus:ring-blue-500" />
    </div>

    <!-- Day Filter Bar -->
    <div class="hidden md:flex items-center space-x-1 bg-zinc-100 p-1 rounded-lg border border-zinc-200" id="dayFilterGroup">
      <button data-day="1" class="day-filter-btn px-2 py-0.5 text-xs font-medium rounded text-zinc-600 hover:bg-white transition">Mon</button>
      <button data-day="2" class="day-filter-btn px-2 py-0.5 text-xs font-medium rounded text-zinc-600 hover:bg-white transition">Tue</button>
      <button data-day="3" class="day-filter-btn px-2 py-0.5 text-xs font-medium rounded text-zinc-600 hover:bg-white transition">Wed</button>
      <button data-day="4" class="day-filter-btn px-2 py-0.5 text-xs font-medium rounded text-zinc-600 hover:bg-white transition">Thu</button>
      <button data-day="5" class="day-filter-btn px-2 py-0.5 text-xs font-medium rounded text-zinc-600 hover:bg-white transition">Fri</button>
      <button data-day="6" class="day-filter-btn px-2 py-0.5 text-xs font-medium rounded text-zinc-600 hover:bg-white transition">Sat</button>
      <button data-day="0" class="day-filter-btn px-2 py-0.5 text-xs font-medium rounded text-zinc-600 hover:bg-white transition">Sun</button>
    </div>

    <!-- Right Actions -->
    <div class="flex items-center space-x-2">
      <!-- Search Input -->
      <div class="relative hidden lg:block">
        <i data-lucide="search" class="w-3.5 h-3.5 text-zinc-400 absolute left-2 top-2"></i>
        <input id="searchInput" type="text" placeholder="Search events..." class="w-36 text-xs pl-7 pr-2 py-1 bg-zinc-50 border border-zinc-300 rounded focus:w-48 transition-all focus:outline-none focus:ring-1 focus:ring-blue-500" />
      </div>

      <!-- Density Zoom Buttons -->
      <div class="hidden sm:flex border border-zinc-300 rounded overflow-hidden">
        <button id="zoomCompact" title="Compact density" class="px-2 py-1 text-xs bg-zinc-50 hover:bg-zinc-100 text-zinc-600 border-r border-zinc-200">Compact</button>
        <button id="zoomNormal" title="Normal density" class="px-2 py-1 text-xs bg-white font-semibold text-blue-600 border-r border-zinc-200">Normal</button>
        <button id="zoomExpanded" title="Expanded density" class="px-2 py-1 text-xs bg-zinc-50 hover:bg-zinc-100 text-zinc-600">Expanded</button>
      </div>

      <!-- Direct Google Sync Button -->
      <button id="googleSyncBtn" class="flex items-center space-x-1.5 px-2.5 py-1 text-xs font-medium rounded bg-blue-50 text-blue-700 border border-blue-300 hover:bg-blue-100 transition">
        <i data-lucide="refresh-cw" id="syncSpinner" class="w-3.5 h-3.5 text-blue-600"></i>
        <span id="syncBtnLabel">Sync Google</span>
      </button>

      <!-- Mobile View Switcher -->
      <button id="mobileViewToggle" class="sm:hidden px-2 py-1 text-xs font-medium border border-zinc-300 rounded bg-white text-zinc-700">
        View Day
      </button>
    </div>
  </header>

  <!-- Sticky Month Banner -->
  <div id="stickyMonthBanner" class="bg-zinc-50/95 backdrop-blur-sm border-b border-zinc-200 text-zinc-600 text-xs font-semibold px-6 py-1.5 flex justify-between items-center z-10">
    <span id="activeMonthLabel">Month Year</span>
    <span id="eventCountBadge" class="text-[11px] text-zinc-400 font-normal">0 events</span>
  </div>

  <!-- Main Viewport Split Screen -->
  <div class="flex-1 flex overflow-hidden relative">

    <!-- LEFT: Continuous Linear Calendar Viewport (~65%) -->
    <main id="linearListContainer" class="w-full sm:w-[65%] h-full overflow-y-auto bg-white relative">
      <div id="calendarRunway" class="divide-y divide-zinc-100"></div>
    </main>

    <!-- RIGHT: Detail & 24h Time-Ruler Inspector (~35%) -->
    <aside id="inspectorPanel" class="hidden sm:flex flex-col w-full sm:w-[35%] h-full bg-zinc-900 text-zinc-100 border-l border-zinc-800 z-10">
      <div class="p-3 border-b border-zinc-800 flex items-center justify-between shrink-0 bg-zinc-900/90">
        <div>
          <h2 id="inspectorDateTitle" class="text-sm font-semibold text-white">Date</h2>
          <p id="inspectorDateSubtitle" class="text-[11px] text-zinc-400">0 events scheduled</p>
        </div>
      </div>

      <!-- Vertical Time Ruler -->
      <div id="timeRulerContainer" class="flex-1 overflow-y-auto relative p-3">
        <div id="rulerCanvas" class="relative h-[1440px] border-l border-zinc-800 ml-12">
          <!-- Two-Hour Markers -->
          <div class="absolute w-full border-t border-zinc-800/80 flex items-center" style="top: 0%"><span class="absolute -left-12 text-[10px] text-zinc-500 font-mono">00:00</span></div>
          <div class="absolute w-full border-t border-zinc-800/40 flex items-center" style="top: 8.33%"><span class="absolute -left-12 text-[10px] text-zinc-500 font-mono">02:00</span></div>
          <div class="absolute w-full border-t border-zinc-800/40 flex items-center" style="top: 16.66%"><span class="absolute -left-12 text-[10px] text-zinc-500 font-mono">04:00</span></div>
          <div class="absolute w-full border-t border-zinc-800 flex items-center" style="top: 25%"><span class="absolute -left-12 text-[10px] text-zinc-500 font-mono">06:00</span></div>
          <div class="absolute w-full border-t border-zinc-800 flex items-center" style="top: 33.33%"><span class="absolute -left-12 text-[10px] text-zinc-500 font-mono">08:00</span></div>
          <div class="absolute w-full border-t border-zinc-800 flex items-center" style="top: 41.66%"><span class="absolute -left-12 text-[10px] text-zinc-500 font-mono">10:00</span></div>
          <div class="absolute w-full border-t border-zinc-800 flex items-center" style="top: 50%"><span class="absolute -left-12 text-[10px] text-zinc-400 font-mono font-medium">12:00</span></div>
          <div class="absolute w-full border-t border-zinc-800 flex items-center" style="top: 58.33%"><span class="absolute -left-12 text-[10px] text-zinc-500 font-mono">14:00</span></div>
          <div class="absolute w-full border-t border-zinc-800 flex items-center" style="top: 66.66%"><span class="absolute -left-12 text-[10px] text-zinc-500 font-mono">16:00</span></div>
          <div class="absolute w-full border-t border-zinc-800 flex items-center" style="top: 75%"><span class="absolute -left-12 text-[10px] text-zinc-500 font-mono">18:00</span></div>
          <div class="absolute w-full border-t border-zinc-800 flex items-center" style="top: 83.33%"><span class="absolute -left-12 text-[10px] text-zinc-500 font-mono">20:00</span></div>
          <div class="absolute w-full border-t border-zinc-800 flex items-center" style="top: 91.66%"><span class="absolute -left-12 text-[10px] text-zinc-500 font-mono">22:00</span></div>
          <div class="absolute w-full border-t border-zinc-800 flex items-center" style="top: 100%"><span class="absolute -left-12 text-[10px] text-zinc-500 font-mono">24:00</span></div>

          <div id="currentTimeIndicator" class="hidden absolute left-0 right-0 border-t border-red-500 z-20 pointer-events-none">
            <span class="absolute -left-12 -top-2 text-[9px] bg-red-600 text-white font-mono px-1 rounded">NOW</span>
          </div>

          <div id="rulerEventsContainer" class="absolute inset-0"></div>
        </div>
      </div>
    </aside>
  </div>

  <!-- Settings Modal (For Client ID) -->
  <div id="setupModal" class="hidden fixed inset-0 bg-black/50 backdrop-blur-sm z-50 flex items-center justify-center p-4">
    <div class="bg-white rounded-xl shadow-2xl w-full max-w-md overflow-hidden border border-zinc-200">
      <div class="p-4 border-b border-zinc-200 flex justify-between items-center bg-zinc-50">
        <h3 class="text-sm font-semibold text-zinc-800">Google Calendar Configuration</h3>
        <button id="closeSetupModalBtn" class="text-zinc-400 hover:text-zinc-700">
          <i data-lucide="x" class="w-4 h-4"></i>
        </button>
      </div>
      <div class="p-4 space-y-3">
        <div>
          <label class="block text-xs font-medium text-zinc-700 mb-1">Your Google OAuth Client ID</label>
          <input type="text" id="gcalClientIdInput" placeholder="xxxx.apps.googleusercontent.com" class="w-full text-xs font-mono px-2.5 py-1.5 border border-zinc-300 rounded focus:ring-1 focus:ring-blue-500 outline-none" />
        </div>
        <p class="text-[11px] text-zinc-500 leading-relaxed">
          Stored strictly in your device's browser. Requests travel exclusively over direct HTTPS to <code>googleapis.com</code>.
        </p>
        <div class="flex justify-end space-x-2 pt-2 border-t border-zinc-100">
          <button type="button" id="saveClientConfigBtn" class="px-3 py-1.5 text-xs font-semibold bg-blue-600 text-white rounded hover:bg-blue-700">Save & Authenticate</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Toast -->
  <div id="toast" class="fixed bottom-4 right-4 bg-zinc-900 text-white text-xs px-3 py-2 rounded-lg shadow-lg transform translate-y-10 opacity-0 transition-all duration-300 z-50 pointer-events-none flex items-center space-x-2">
    <span id="toastMessage">Synced</span>
  </div>

  <script>
    const CLIENT_ID_STORAGE_KEY = 'continuous_gcal_client_id';
    const EVENTS_STORAGE_KEY = 'continuous_gcal_cached_events';

    const state = {
      events: [],
      selectedDate: getFormattedDate(new Date()),
      density: 'normal',
      activeDayFilter: null,
      searchQuery: '',
      tokenClient: null,
      accessToken: null
    };

    function getFormattedDate(d) {
      const year = d.getFullYear();
      const month = String(d.getMonth() + 1).padStart(2, '0');
      const day = String(d.getDate()).padStart(2, '0');
      return `${year}-${month}-${day}`;
    }

    function showToast(msg) {
      const toast = document.getElementById('toast');
      const toastMsg = document.getElementById('toastMessage');
      toastMsg.textContent = msg;
      toast.classList.remove('translate-y-10', 'opacity-0');
      setTimeout(() => {
        toast.classList.add('translate-y-10', 'opacity-0');
      }, 2500);
    }

    /* -------------------------------------------------------------
     * Google Identity Services & Direct Calendar API Fetching
     * ----------------------------------------------------------- */
    function initGoogleClient(clientId) {
      if (!clientId || !window.google) return;
      try {
        state.tokenClient = google.accounts.oauth2.initTokenClient({
          client_id: clientId,
          scope: 'https://www.googleapis.com/auth/calendar.events.readonly',
          callback: async (resp) => {
            if (resp.error) {
              showToast('Google login cancelled');
              return;
            }
            state.accessToken = resp.access_token;
            await fetchGoogleEvents();
          }
        });
      } catch (err) {
        console.error(err);
      }
    }

    async function fetchGoogleEvents() {
      if (!state.accessToken) return;

      const spinner = document.getElementById('syncSpinner');
      spinner.classList.add('animate-spin');

      try {
        // Fetch 30 days back to 180 days forward directly from Google APIs
        const now = new Date();
        const timeMin = new Date(now.setDate(now.getDate() - 30)).toISOString();
        const timeMax = new Date(now.setDate(now.getDate() + 210)).toISOString();

        const url = `https://www.googleapis.com/calendar/v3/calendars/primary/events?singleEvents=true&orderBy=startTime&timeMin=${timeMin}&timeMax=${timeMax}&maxResults=250`;
        
        const response = await fetch(url, {
          headers: { Authorization: `Bearer ${state.accessToken}` }
        });

        if (!response.ok) throw new Error('Failed to fetch from Google');
        const data = await response.json();

        state.events = (data.items || []).map(item => {
          let startDate = '', startTime = '09:00', endTime = '10:00';
          
          if (item.start.dateTime) {
            const startD = new Date(item.start.dateTime);
            const endD = new Date(item.end.dateTime);
            startDate = getFormattedDate(startD);
            startTime = `${String(startD.getHours()).padStart(2,'0')}:${String(startD.getMinutes()).padStart(2,'0')}`;
            endTime = `${String(endD.getHours()).padStart(2,'0')}:${String(endD.getMinutes()).padStart(2,'0')}`;
          } else if (item.start.date) {
            startDate = item.start.date;
            startTime = '09:00';
            endTime = '10:00';
          }

          return {
            id: item.id,
            title: item.summary || '(No title)',
            startDate,
            startTime,
            endTime,
            description: item.description || ''
          };
        });

        localStorage.setItem(EVENTS_STORAGE_KEY, JSON.stringify(state.events));
        renderLinearList();
        renderTimeRuler();
        showToast(`Synced ${state.events.length} events directly from Google`);
      } catch (err) {
        console.error(err);
        showToast('Google sync error');
      } finally {
        spinner.classList.remove('animate-spin');
      }
    }

    /* -------------------------------------------------------------
     * Linear Calendar Runway Construction
     * ----------------------------------------------------------- */
    const DAYS_BEFORE = 45;
    const DAYS_AFTER = 180;
    const runwayDates = [];

    function generateRunwayDates() {
      runwayDates.length = 0;
      const today = new Date();
      for (let i = -DAYS_BEFORE; i <= DAYS_AFTER; i++) {
        const d = new Date();
        d.setDate(today.getDate() + i);
        runwayDates.push(new Date(d));
      }
    }

    function getWeekNumber(date) {
      const d = new Date(Date.UTC(date.getFullYear(), date.getMonth(), date.getDate()));
      const dayNum = d.getUTCDay() || 7;
      d.setUTCDate(d.getUTCDate() + 4 - dayNum);
      const yearStart = new Date(Date.UTC(d.getUTCFullYear(),0,1));
      return Math.ceil((((d - yearStart) / 86400000) + 1)/7);
    }

    const dayNames = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
    const monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
    const fullMonthNames = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];

    function renderLinearList() {
      const container = document.getElementById('calendarRunway');
      container.innerHTML = '';
      const todayStr = getFormattedDate(new Date());

      runwayDates.forEach(date => {
        const dateStr = getFormattedDate(date);
        const dayOfWeek = date.getDay();
        const isMonday = dayOfWeek === 1;
        const isWeekend = dayOfWeek === 0 || dayOfWeek === 6;
        const isSelected = dateStr === state.selectedDate;
        const isToday = dateStr === todayStr;

        const row = document.createElement('div');
        row.id = `day-row-${dateStr}`;
        row.dataset.date = dateStr;
        row.dataset.day = dayOfWeek;

        let rowClass = 'flex items-center group cursor-pointer transition border-b border-zinc-100 hover:bg-blue-50/30 ';
        if (state.density === 'compact') rowClass += 'h-8 text-xs ';
        else if (state.density === 'expanded') rowClass += 'min-h-[52px] py-1 text-xs ';
        else rowClass += 'h-11 text-xs ';

        if (isWeekend) rowClass += 'bg-zinc-50/70 ';
        if (isSelected) rowClass += 'bg-blue-50/60 ';

        row.className = rowClass;

        // Week marker
        const col1 = document.createElement('div');
        col1.className = 'w-14 sm:w-16 shrink-0 pl-3 font-mono text-[10px] text-zinc-400 select-none';
        if (isMonday) {
          col1.innerHTML = `<span class="font-semibold text-zinc-600">W${String(getWeekNumber(date)).padStart(2, '0')}</span>`;
        }

        // Date badge
        const col2 = document.createElement('div');
        col2.className = 'w-24 sm:w-28 shrink-0 flex items-center space-x-1.5 pl-1';

        const dayBadge = document.createElement('div');
        let badgeClass = 'px-1.5 py-0.5 rounded font-mono text-[11px] flex items-center space-x-1 ';
        if (isSelected) badgeClass += 'bg-blue-600 text-white font-semibold ';
        else if (isToday) badgeClass += 'bg-blue-100 text-blue-800 font-bold border border-blue-300 ';
        else if (isWeekend) badgeClass += 'text-zinc-500 font-medium ';
        else badgeClass += 'text-zinc-700 font-medium ';

        if (state.activeDayFilter !== null && Number(state.activeDayFilter) === dayOfWeek) {
          badgeClass += 'day-highlight ';
        }

        dayBadge.className = badgeClass;
        dayBadge.innerHTML = `<span>${monthNames[date.getMonth()]} ${String(date.getDate()).padStart(2, '0')}</span> <span class="opacity-75">${dayNames[dayOfWeek]}</span>`;
        col2.appendChild(dayBadge);

        // Events horizon
        const col3 = document.createElement('div');
        col3.className = 'flex-1 overflow-x-auto overflow-y-hidden flex items-center space-x-1.5 px-2 no-scrollbar';

        const dayEvents = state.events
          .filter(e => e.startDate === dateStr)
          .sort((a, b) => a.startTime.localeCompare(b.startTime));

        dayEvents.forEach(evt => {
          if (state.searchQuery && !evt.title.toLowerCase().includes(state.searchQuery.toLowerCase())) return;

          const chip = document.createElement('div');
          chip.className = 'flex items-center shrink-0 border rounded text-[11px] shadow-sm bg-white overflow-hidden';
          chip.innerHTML = `
            <span class="bg-blue-600 text-white text-[10px] font-mono px-1.5 py-0.5">${evt.startTime}</span>
            <span class="px-2 py-0.5 text-zinc-800 font-medium truncate max-w-[140px] sm:max-w-[200px]">${escapeHtml(evt.title)}</span>
          `;
          col3.appendChild(chip);
        });

        row.addEventListener('click', () => {
          state.selectedDate = dateStr;
          renderLinearList();
          renderTimeRuler();
        });

        row.appendChild(col1);
        row.appendChild(col2);
        row.appendChild(col3);
        container.appendChild(row);
      });

      updateStickyMonthBanner();
    }

    function escapeHtml(str) {
      return (str || '').replace(/[&<>'"]/g, tag => ({
        '&': '&amp;',
        '<': '&lt;',
        '>': '&gt;',
        "'": '&#39;',
        '"': '&quot;'
      }[tag] || tag));
    }

    function updateStickyMonthBanner() {
      const container = document.getElementById('linearListContainer');
      const scrollTop = container.scrollTop;
      const rows = container.querySelectorAll('[data-date]');

      for (let row of rows) {
        if (row.offsetTop + row.clientHeight > scrollTop) {
          const dStr = row.dataset.date;
          const [y, m] = dStr.split('-');
          const monthIdx = parseInt(m, 10) - 1;
          document.getElementById('activeMonthLabel').textContent = `${fullMonthNames[monthIdx]} ${y}`;
          break;
        }
      }
      document.getElementById('eventCountBadge').textContent = `${state.events.length} total events`;
    }

    /* -------------------------------------------------------------
     * Right Area: Time Ruler
     * ----------------------------------------------------------- */
    function renderTimeRuler() {
      const selected = new Date(state.selectedDate + 'T00:00:00');
      const dayIndex = selected.getDay();
      const formattedTitle = `${dayNames[dayIndex]}, ${selected.getDate()} ${fullMonthNames[selected.getMonth()]} ${selected.getFullYear()}`;
      document.getElementById('inspectorDateTitle').textContent = formattedTitle;

      const eventsToday = state.events.filter(e => e.startDate === state.selectedDate);
      document.getElementById('inspectorDateSubtitle').textContent = `${eventsToday.length} appointment${eventsToday.length === 1 ? '' : 's'} scheduled`;

      const container = document.getElementById('rulerEventsContainer');
      container.innerHTML = '';

      const todayStr = getFormattedDate(new Date());
      const nowIndicator = document.getElementById('currentTimeIndicator');
      if (state.selectedDate === todayStr) {
        const now = new Date();
        const mins = now.getHours() * 60 + now.getMinutes();
        const topPct = (mins / 1440) * 100;
        nowIndicator.style.top = `${topPct}%`;
        nowIndicator.classList.remove('hidden');
      } else {
        nowIndicator.classList.add('hidden');
      }

      eventsToday.forEach(evt => {
        const startMins = parseTimeToMinutes(evt.startTime);
        const endMins = parseTimeToMinutes(evt.endTime);
        const durationMins = Math.max(endMins - startMins, 25);

        const topPercent = (startMins / 1440) * 100;
        const heightPercent = (durationMins / 1440) * 100;

        const card = document.createElement('div');
        card.className = 'absolute left-2 right-4 rounded-md bg-zinc-800 border border-zinc-700 p-2 shadow-md overflow-hidden flex flex-col justify-between';
        card.style.top = `${topPercent}%`;
        card.style.height = `max(34px, ${heightPercent}%)`;

        card.innerHTML = `
          <div>
            <div class="flex items-center justify-between">
              <span class="text-xs font-semibold text-white truncate">${escapeHtml(evt.title)}</span>
              <span class="text-[10px] text-zinc-400 font-mono">${evt.startTime} - ${evt.endTime}</span>
            </div>
            ${evt.description ? `<p class="text-[11px] text-zinc-400 line-clamp-2 mt-0.5">${escapeHtml(evt.description)}</p>` : ''}
          </div>
        `;
        container.appendChild(card);
      });
    }

    function parseTimeToMinutes(t) {
      if (!t) return 0;
      const [h, m] = t.split(':').map(Number);
      return h * 60 + m;
    }

    function scrollToDate(dateStr) {
      const targetRow = document.getElementById(`day-row-${dateStr}`);
      if (targetRow) {
        const container = document.getElementById('linearListContainer');
        container.scrollTo({ top: targetRow.offsetTop - 40, behavior: 'smooth' });
      }
    }

    document.getElementById('todayBtn').addEventListener('click', () => {
      const todayStr = getFormattedDate(new Date());
      state.selectedDate = todayStr;
      renderLinearList();
      renderTimeRuler();
      scrollToDate(todayStr);
    });

    document.getElementById('jumpDateInput').addEventListener('change', (e) => {
      if (e.target.value) {
        state.selectedDate = e.target.value;
        renderLinearList();
        renderTimeRuler();
        scrollToDate(e.target.value);
      }
    });

    document.querySelectorAll('.day-filter-btn').forEach(btn => {
      btn.addEventListener('click', () => {
        const day = btn.dataset.day;
        if (state.activeDayFilter === day) {
          state.activeDayFilter = null;
          btn.classList.remove('bg-amber-100', 'text-amber-800', 'font-bold');
        } else {
          document.querySelectorAll('.day-filter-btn').forEach(b => b.classList.remove('bg-amber-100', 'text-amber-800', 'font-bold'));
          state.activeDayFilter = day;
          btn.classList.add('bg-amber-100', 'text-amber-800', 'font-bold');
        }
        renderLinearList();
      });
    });

    const zoomCompact = document.getElementById('zoomCompact');
    const zoomNormal = document.getElementById('zoomNormal');
    const zoomExpanded = document.getElementById('zoomExpanded');

    function setDensity(d) {
      state.density = d;
      [zoomCompact, zoomNormal, zoomExpanded].forEach(btn => {
        btn.classList.remove('bg-white', 'font-semibold', 'text-blue-600');
        btn.classList.add('bg-zinc-50', 'text-zinc-600');
      });
      if (d === 'compact') zoomCompact.classList.add('bg-white', 'font-semibold', 'text-blue-600');
      if (d === 'normal') zoomNormal.classList.add('bg-white', 'font-semibold', 'text-blue-600');
      if (d === 'expanded') zoomExpanded.classList.add('bg-white', 'font-semibold', 'text-blue-600');
      renderLinearList();
    }

    zoomCompact.addEventListener('click', () => setDensity('compact'));
    zoomNormal.addEventListener('click', () => setDensity('normal'));
    zoomExpanded.addEventListener('click', () => setDensity('expanded'));

    document.getElementById('searchInput').addEventListener('input', (e) => {
      state.searchQuery = e.target.value.trim();
      renderLinearList();
    });

    let mobileShowInspector = false;
    document.getElementById('mobileViewToggle').addEventListener('click', () => {
      mobileShowInspector = !mobileShowInspector;
      const list = document.getElementById('linearListContainer');
      const insp = document.getElementById('inspectorPanel');
      const btn = document.getElementById('mobileViewToggle');

      if (mobileShowInspector) {
        list.classList.add('hidden');
        insp.classList.remove('hidden');
        insp.classList.add('flex');
        btn.textContent = 'View List';
      } else {
        list.classList.remove('hidden');
        insp.classList.add('hidden');
        insp.classList.remove('flex');
        btn.textContent = 'View Day';
      }
    });

    // Google Sync Button Handler
    document.getElementById('googleSyncBtn').addEventListener('click', () => {
      const savedClientId = localStorage.getItem(CLIENT_ID_STORAGE_KEY);
      if (!savedClientId) {
        document.getElementById('setupModal').classList.remove('hidden');
      } else {
        if (!state.tokenClient) initGoogleClient(savedClientId);
        state.tokenClient.requestAccessToken({ prompt: '' });
      }
    });

    document.getElementById('saveClientConfigBtn').addEventListener('click', () => {
      const val = document.getElementById('gcalClientIdInput').value.trim();
      if (!val) return;
      localStorage.setItem(CLIENT_ID_STORAGE_KEY, val);
      document.getElementById('setupModal').classList.add('hidden');
      initGoogleClient(val);
      state.tokenClient.requestAccessToken({ prompt: 'consent' });
    });

    document.getElementById('closeSetupModalBtn').addEventListener('click', () => {
      document.getElementById('setupModal').classList.add('hidden');
    });

    document.getElementById('linearListContainer').addEventListener('scroll', updateStickyMonthBanner);

    window.addEventListener('DOMContentLoaded', () => {
      lucide.createIcons();
      generateRunwayDates();

      // Load cached events from local storage
      const cached = localStorage.getItem(EVENTS_STORAGE_KEY);
      if (cached) {
        try { state.events = JSON.parse(cached); } catch(e) {}
      }

      renderLinearList();
      renderTimeRuler();

      const savedClientId = localStorage.getItem(CLIENT_ID_STORAGE_KEY);
      if (savedClientId) {
        document.getElementById('gcalClientIdInput').value = savedClientId;
        setTimeout(() => initGoogleClient(savedClientId), 300);
      }

      setTimeout(() => {
        const todayStr = getFormattedDate(new Date());
        scrollToDate(todayStr);
      }, 50);
    });
  </script>
</body>
</html>
