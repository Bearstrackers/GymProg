<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Workout Dashboard — 12-Week Program</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500;600;700&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
<style>
:root {
    --bg: #08080c; --bg-elevated: #0e0e14; --card: #13131c; --card-hover: #1a1a26;
    --border: #1e1e2c; --border-light: #2d2d3d; --text: #eaeaf0; --text-secondary: #a0a0b8; --text-muted: #5a5a72;
    --accent: #c8ff00; --accent-dim: rgba(200,255,0,0.08); --accent-glow: rgba(200,255,0,0.15);
    --red: #ff4d6a; --cyan: #00e5ff; --amber: #ffab00; --green: #00e676; --pink: #ff80ab; --lavender: #b388ff; --coral: #ff6b6b; --orange: #ff9100;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
html { scroll-behavior: smooth; }
body { font-family: 'Space Grotesk', sans-serif; background: var(--bg); color: var(--text); min-height: 100vh; overflow-x: hidden; }
.font-mono { font-family: 'JetBrains Mono', monospace; }

.blob { position: absolute; border-radius: 50%; filter: blur(100px); pointer-events: none; }
.blob-1 { width: 500px; height: 500px; background: var(--accent); opacity: 0.04; top: -150px; left: -100px; animation: floatBlob 25s ease-in-out infinite; }
.blob-2 { width: 400px; height: 400px; background: var(--cyan); opacity: 0.035; bottom: 10%; right: -80px; animation: floatBlob 30s ease-in-out infinite reverse; }
.blob-3 { width: 600px; height: 600px; background: var(--coral); opacity: 0.02; top: 50%; left: 30%; animation: floatBlob 35s ease-in-out infinite 5s; }
@keyframes floatBlob { 0%,100%{transform:translate(0,0)scale(1)} 25%{transform:translate(40px,-30px)scale(1.05)} 50%{transform:translate(-20px,50px)scale(0.95)} 75%{transform:translate(30px,20px)scale(1.02)} }

.dot-grid { position: fixed; inset: 0; z-index: 0; pointer-events: none; background-image: radial-gradient(rgba(255,255,255,0.03) 1px, transparent 1px); background-size: 28px 28px; }

.card { background: var(--card); border: 1px solid var(--border); border-radius: 14px; padding: 1.25rem; transition: all 0.35s cubic-bezier(0.25, 0.46, 0.45, 0.94); opacity: 0; transform: translateY(18px); animation: cardIn 0.55s ease forwards; }
.card:hover { border-color: var(--border-light); background: var(--card-hover); box-shadow: 0 0 30px var(--accent-dim), 0 8px 32px rgba(0,0,0,0.3); transform: translateY(-2px); }
@keyframes cardIn { to { opacity: 1; transform: translateY(0); } }

.metric-icon { width: 36px; height: 36px; border-radius: 10px; display: flex; align-items: center; justify-content: center; font-size: 14px; flex-shrink: 0; }
.metric-val { font-family: 'JetBrains Mono', monospace; font-weight: 700; font-size: 1.75rem; line-height: 1.1; color: var(--text); }

.phase-track { height: 6px; border-radius: 3px; background: var(--border); position: relative; overflow: visible; }
.phase-fill { height: 100%; border-radius: 3px; position: absolute; top: 0; left: 0; transition: width 0.8s ease; }
.phase-marker { width: 14px; height: 14px; border-radius: 50%; background: var(--accent); border: 3px solid var(--bg); position: absolute; top: 50%; transform: translate(-50%, -50%); box-shadow: 0 0 12px var(--accent-glow), 0 0 24px var(--accent-dim); z-index: 2; animation: markerPulse 2.5s ease-in-out infinite; }
@keyframes markerPulse { 0%,100%{box-shadow:0 0 12px var(--accent-glow),0 0 24px var(--accent-dim)} 50%{box-shadow:0 0 20px var(--accent-glow),0 0 40px var(--accent-dim)} }

.gradient-line { height: 2px; background: linear-gradient(90deg, var(--accent), var(--cyan), var(--lavender), transparent); border-radius: 1px; margin-top: 1rem; animation: shimmer 4s ease-in-out infinite alternate; background-size: 200% 100%; }
@keyframes shimmer { 0%{background-position:0% 50%} 100%{background-position:100% 50%} }

.matrix-grid { display: grid; grid-template-columns: 36px repeat(6, 1fr); gap: 3px; }
.matrix-cell { aspect-ratio: 1; border-radius: 4px; transition: all 0.2s ease; cursor: default; position: relative; }
.matrix-cell:hover { transform: scale(1.2); z-index: 5; }
.matrix-cell.completed { background: var(--accent); opacity: 0.8; }
.matrix-cell.completed:hover { opacity: 1; }
.matrix-cell.current { background: var(--accent-dim); border: 1px solid rgba(200,255,0,0.2); }
.matrix-cell.missed { background: rgba(255,77,106,0.08); border: 1px solid rgba(255,77,106,0.1); }
.matrix-cell.future { background: var(--border); opacity: 0.3; }
.matrix-cell.future:hover { opacity: 0.5; }
.matrix-cell.today { background: var(--accent); opacity: 1; box-shadow: 0 0 8px var(--accent-glow); animation: todayPulse 2s ease-in-out infinite; }
@keyframes todayPulse { 0%,100%{opacity:1} 50%{opacity:0.6} }
.matrix-cell[data-tip]:hover::after { content: attr(data-tip); position: absolute; bottom: calc(100% + 6px); left: 50%; transform: translateX(-50%); background: #1a1a26; color: var(--text); font-size: 11px; padding: 4px 8px; border-radius: 6px; border: 1px solid var(--border-light); white-space: nowrap; z-index: 100; pointer-events: none; font-family: 'Space Grotesk', sans-serif; }

.lifts-table { width: 100%; border-collapse: separate; border-spacing: 0; }
.lifts-table thead th { text-align: left; padding: 0.6rem 0.75rem; font-size: 0.7rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.08em; color: var(--text-muted); border-bottom: 1px solid var(--border); }
.lifts-table tbody td { padding: 0.65rem 0.75rem; font-size: 0.85rem; border-bottom: 1px solid var(--border); color: var(--text-secondary); }
.lifts-table tbody tr:last-child td { border-bottom: none; }
.lifts-table tbody tr { transition: background 0.2s; }
.lifts-table tbody tr:hover { background: var(--accent-dim); }

.pr-item { display: flex; align-items: center; justify-content: space-between; padding: 0.6rem 0; border-bottom: 1px solid var(--border); }
.pr-item:last-child { border-bottom: none; }
.stat-row { display: flex; align-items: center; justify-content: space-between; padding: 0.55rem 0; border-bottom: 1px solid var(--border); }
.stat-row:last-child { border-bottom: none; }
.ring-bg { stroke: var(--border); }
.ring-fill { stroke: var(--accent); stroke-linecap: round; transition: stroke-dasharray 1.2s ease; filter: drop-shadow(0 0 4px var(--accent-glow)); }
.chart-wrap { position: relative; }
.chart-wrap canvas { max-width: 100%; }
.empty-state { position: absolute; inset: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; color: var(--text-muted); font-size: 0.85rem; gap: 0.5rem; pointer-events: none; }
.empty-state i { font-size: 1.5rem; opacity: 0.4; }
.doughnut-center { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); text-align: center; pointer-events: none; }
::-webkit-scrollbar { width: 6px; }
::-webkit-scrollbar-track { background: var(--bg); }
::-webkit-scrollbar-thumb { background: var(--border-light); border-radius: 3px; }
.section-title { font-size: 0.75rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.1em; color: var(--text-muted); margin-bottom: 1rem; }
.badge { font-size: 0.6rem; font-weight: 700; padding: 2px 6px; border-radius: 4px; text-transform: uppercase; letter-spacing: 0.05em; }
.loading-overlay { position: fixed; inset: 0; z-index: 1000; background: var(--bg); display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 1rem; transition: opacity 0.5s ease; }
.loading-overlay.hidden { opacity: 0; pointer-events: none; }
.spinner { width: 40px; height: 40px; border: 3px solid var(--border); border-top-color: var(--accent); border-radius: 50%; animation: spin 1s linear infinite; }
@keyframes spin { to { transform: rotate(360deg); } }
.error-state { text-align: center; padding: 3rem 1rem; color: var(--text-muted); }
.error-state i { font-size: 2rem; color: var(--red); margin-bottom: 1rem; }
.setup-state { text-align: center; padding: 3rem 1rem; color: var(--text-muted); max-width: 600px; margin: 0 auto; }
.setup-state i { font-size: 3rem; color: var(--accent); margin-bottom: 1.5rem; }
.volume-bar { width: 100%; height: 4px; background: var(--border); border-radius: 2px; overflow: hidden; margin-top: 4px; }
.volume-fill { height: 100%; background: linear-gradient(90deg, var(--accent), var(--cyan)); border-radius: 2px; transition: width 0.8s ease; }
.hidden { display: none !important; }
.code-block { background: var(--bg-elevated); border: 1px solid var(--border); border-radius: 8px; padding: 12px 16px; font-family: 'JetBrains Mono', monospace; font-size: 12px; color: var(--accent); word-break: break-all; margin: 12px 0; }

@media (prefers-reduced-motion: reduce) { *,*::before,*::after { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; } }
@media (max-width: 768px) { .metric-val { font-size: 1.4rem; } }
</style>
</head>
<body>
<div class="loading-overlay" id="loadingOverlay"><div class="spinner"></div><p style="color:var(--text-muted); font-size:0.85rem;">Loading workout data...</p></div>
<div class="fixed inset-0 z-0 overflow-hidden"><div class="blob blob-1"></div><div class="blob blob-2"></div><div class="blob blob-3"></div></div>
<div class="dot-grid"></div>

<main class="relative z-10 max-w-7xl mx-auto px-4 sm:px-6 py-6 sm:py-10">
<header class="mb-6" style="animation-delay:0s">
<div class="flex flex-col sm:flex-row sm:items-end sm:justify-between gap-2">
<div>
<h1 class="text-2xl sm:text-3xl font-bold tracking-tight" style="color:var(--text)">WORKOUT DASHBOARD</h1>
<p class="text-sm mt-1" style="color:var(--text-muted)">12-Week Program &bull; <span id="currentDate"></span> &bull; <span id="currentPhase">Phase 1</span></p>
</div>
<div class="flex items-center gap-3">
<button onclick="refreshData()" class="px-3 py-1.5 rounded-lg text-xs font-medium transition-all hover:opacity-80" style="background:var(--card); border:1px solid var(--border); color:var(--text-secondary); cursor:pointer;"><i class="fa-solid fa-rotate-right mr-1"></i> Refresh</button>
<p class="text-xs font-mono" style="color:var(--text-muted)"><i class="fa-regular fa-clock mr-1"></i> <span id="dataSource">Live from Sheets</span></p>
</div>
</div>
<div class="gradient-line"></div>
</header>

<!-- SETUP REQUIRED STATE -->
<div id="setupState" class="setup-state hidden">
<i class="fa-solid fa-gear"></i>
<h3 class="text-xl font-bold mb-3" style="color:var(--text)">Dashboard Setup Required</h3>
<p class="text-sm mb-4">This dashboard needs to connect to your Google Sheets data via a Google Apps Script Web App URL.</p>
<div style="text-align:left; background:var(--card); border:1px solid var(--border); border-radius:12px; padding:1.5rem; margin:1rem 0;">
<h4 class="font-semibold mb-3" style="color:var(--accent)">Step 1: Get Your Web App URL</h4>
<ol style="margin-left:1.2rem; line-height:1.8; font-size:13px; color:var(--text-secondary);">
<li>Open your Google Sheet with workout data</li>
<li>Go to <strong>Extensions &rarr; Apps Script</strong></li>
<li>Paste the backend code (from apps-script-backend.js)</li>
<li>Click <strong>Deploy &rarr; New deployment</strong></li>
<li>Select <strong>Web app</strong>, set access to <strong>Anyone</strong></li>
<li>Copy the URL that looks like:</li>
</ol>
<div class="code-block">https://script.google.com/macros/s/ABC123/exec</div>
<h4 class="font-semibold mb-3 mt-4" style="color:var(--accent)">Step 2: Update This File</h4>
<p style="font-size:13px; color:var(--text-secondary); line-height:1.6;">Open this HTML file in a text editor, find the line with <code style="background:var(--bg-elevated); padding:2px 6px; border-radius:4px;">API_URL</code> near the top, and replace it with your URL.</p>
</div>
</div>

<!-- ERROR STATE -->
<div id="errorState" class="error-state hidden">
<i class="fa-solid fa-triangle-exclamation"></i>
<h3 class="text-lg font-semibold mb-2" style="color:var(--text)">Could not load data</h3>
<p class="text-sm mb-4" id="errorMessage">Check your API_URL and make sure the Google Sheet is shared.</p>
<button onclick="location.reload()" class="px-4 py-2 rounded-lg text-sm font-medium" style="background:var(--accent); color:var(--bg); border:none; cursor:pointer;">Try Again</button>
</div>

<!-- DASHBOARD CONTENT -->
<div id="dashboardContent">
<section class="card mb-5" style="animation-delay:0.05s; padding:1rem 1.25rem">
<div class="flex items-center justify-between mb-2"><span class="text-xs font-semibold uppercase tracking-widest" style="color:var(--text-muted)">Program Timeline</span><span class="text-xs font-mono" style="color:var(--accent)" id="weekIndicator">Week 1 of 12</span></div>
<div class="phase-track">
<div class="phase-fill" style="width:33.33%; background: linear-gradient(90deg, var(--accent), rgba(200,255,0,0.4));"></div>
<div class="phase-fill" style="width:33.33%; left:33.33%; background: linear-gradient(90deg, rgba(200,255,0,0.15), rgba(0,229,255,0.15));"></div>
<div class="phase-fill" style="width:33.34%; left:66.66%; background: linear-gradient(90deg, rgba(0,229,255,0.08), rgba(179,136,255,0.08));"></div>
<div class="phase-marker" id="phaseMarker" style="left:4.2%"></div>
</div>
<div class="flex justify-between mt-2">
<div class="text-center"><div class="text-[10px] font-semibold uppercase tracking-wider" style="color:var(--accent)">Phase 1</div><div class="text-[10px]" style="color:var(--text-muted)">Wk 1-4 &bull; Hypertrophy</div></div>
<div class="text-center"><div class="text-[10px] font-semibold uppercase tracking-wider" style="color:var(--text-muted)">Phase 2</div><div class="text-[10px]" style="color:var(--text-muted)">Wk 5-8 &bull; Strength</div></div>
<div class="text-center"><div class="text-[10px] font-semibold uppercase tracking-wider" style="color:var(--text-muted)">Phase 3</div><div class="text-[10px]" style="color:var(--text-muted)">Wk 9-12 &bull; Peaking</div></div>
</div>
</section>

<section class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-3 sm:gap-4 mb-5">
<div class="card" style="animation-delay:0.08s"><div class="flex items-center gap-2.5 mb-3"><div class="metric-icon" style="background:var(--accent-dim); color:var(--accent)"><i class="fa-solid fa-dumbbell"></i></div><span class="text-xs font-medium" style="color:var(--text-muted)">Sessions</span></div><div class="metric-val" id="metricSessions">0</div><div class="text-[11px] mt-1.5" style="color:var(--text-muted)">of <span id="metricTotalSessions">72</span> possible</div><div class="volume-bar"><div class="volume-fill" id="sessionsProgress" style="width:0%"></div></div></div>
<div class="card" style="animation-delay:0.11s"><div class="flex items-center gap-2.5 mb-3"><div class="metric-icon" style="background:rgba(0,230,118,0.1); color:var(--green)"><i class="fa-solid fa-bullseye"></i></div><span class="text-xs font-medium" style="color:var(--text-muted)">Adherence</span></div><div class="flex items-end gap-2"><svg width="48" height="48" viewBox="0 0 72 72" class="flex-shrink-0"><circle cx="36" cy="36" r="28" fill="none" stroke-width="5" class="ring-bg"/><circle cx="36" cy="36" r="28" fill="none" stroke-width="5" class="ring-fill" id="adherenceRing" stroke-dasharray="0 175.93" transform="rotate(-90 36 36)"/></svg><div class="metric-val text-lg" id="metricAdherence">0%</div></div><div class="text-[11px] mt-1" style="color:var(--text-muted)">Completion rate</div></div>
<div class="card" style="animation-delay:0.14s"><div class="flex items-center gap-2.5 mb-3"><div class="metric-icon" style="background:rgba(255,107,107,0.1); color:var(--coral)"><i class="fa-solid fa-fire-flame-curved"></i></div><span class="text-xs font-medium" style="color:var(--text-muted)">Avg RPE</span></div><div class="metric-val" id="metricRPE" style="color:var(--text-muted); font-size:1.4rem">-</div><div class="text-[11px] mt-1.5" style="color:var(--text-muted)">Across all logged sets</div></div>
<div class="card" style="animation-delay:0.17s"><div class="flex items-center gap-2.5 mb-3"><div class="metric-icon" style="background:rgba(0,229,255,0.1); color:var(--cyan)"><i class="fa-solid fa-weight-hanging"></i></div><span class="text-xs font-medium" style="color:var(--text-muted)">Tonnage</span></div><div class="metric-val text-lg" id="metricTonnage">0 kg</div><div class="text-[11px] mt-1.5" style="color:var(--text-muted)">Total weight x reps</div></div>
<div class="card" style="animation-delay:0.2s"><div class="flex items-center gap-2.5 mb-3"><div class="metric-icon" style="background:rgba(255,171,0,0.1); color:var(--amber)"><i class="fa-solid fa-trophy"></i></div><span class="text-xs font-medium" style="color:var(--text-muted)">Best 1RM</span></div><div class="metric-val text-lg" id="metricBest1RM">-</div><div class="text-[11px] mt-1.5" style="color:var(--text-muted)" id="metricBest1RMLabel">-</div></div>
<div class="card" style="animation-delay:0.23s"><div class="flex items-center gap-2.5 mb-3"><div class="metric-icon" style="background:rgba(179,136,255,0.1); color:var(--lavender)"><i class="fa-solid fa-layer-group"></i></div><span class="text-xs font-medium" style="color:var(--text-muted)">Total Sets</span></div><div class="metric-val" id="metricSets">0</div><div class="text-[11px] mt-1.5" style="color:var(--text-muted)">Across all sessions</div></div>
</section>

<section class="grid grid-cols-1 lg:grid-cols-3 gap-4 mb-5">
<div class="card lg:col-span-2" style="animation-delay:0.26s"><div class="flex items-center justify-between mb-4 flex-wrap gap-2"><h3 class="section-title mb-0">1RM Progression</h3><div class="flex gap-3 flex-wrap" id="ormLegend"></div></div><div class="chart-wrap" style="height:260px"><canvas id="ormChart"></canvas></div></div>
<div class="card" style="animation-delay:0.29s"><h3 class="section-title">Session Completion</h3><div class="matrix-grid mb-1"><div></div><div class="text-[9px] text-center font-semibold uppercase" style="color:var(--text-muted)">D1</div><div class="text-[9px] text-center font-semibold uppercase" style="color:var(--text-muted)">D2</div><div class="text-[9px] text-center font-semibold uppercase" style="color:var(--text-muted)">D3</div><div class="text-[9px] text-center font-semibold uppercase" style="color:var(--text-muted)">D4</div><div class="text-[9px] text-center font-semibold uppercase" style="color:var(--text-muted)">D5</div><div class="text-[9px] text-center font-semibold uppercase" style="color:var(--text-muted)">D6</div></div><div id="sessionMatrix" class="matrix-grid"></div><div class="flex items-center gap-3 mt-3 pt-3" style="border-top:1px solid var(--border)"><span class="flex items-center gap-1.5 text-[10px]" style="color:var(--text-muted)"><span class="inline-block w-3 h-3 rounded" style="background:var(--accent); opacity:0.8"></span>Done</span><span class="flex items-center gap-1.5 text-[10px]" style="color:var(--text-muted)"><span class="inline-block w-3 h-3 rounded" style="background:var(--accent-dim); border:1px solid rgba(200,255,0,0.2)"></span>This Week</span><span class="flex items-center gap-1.5 text-[10px]" style="color:var(--text-muted)"><span class="inline-block w-3 h-3 rounded" style="background:var(--border); opacity:0.3"></span>Upcoming</span></div></div>
</section>

<section class="grid grid-cols-1 lg:grid-cols-2 gap-4 mb-5">
<div class="card" style="animation-delay:0.32s"><h3 class="section-title">Weekly Training Volume (Sets)</h3><div class="chart-wrap" style="height:240px"><canvas id="volumeChart"></canvas></div></div>
<div class="card" style="animation-delay:0.35s; position:relative"><h3 class="section-title">RPE Trends (Avg per Week)</h3><div class="chart-wrap" style="height:240px"><canvas id="rpeChart"></canvas><div class="empty-state" id="rpeEmptyState"><i class="fa-regular fa-face-meh"></i><span>No RPE data logged yet</span><span class="text-[11px]" style="color:var(--border-light)">Fill in RPE values on your day sheets</span></div></div></div>
</section>

<section class="card mb-5" style="animation-delay:0.38s; padding:1rem 1.25rem; overflow-x:auto"><div class="flex items-center justify-between mb-4 flex-wrap gap-2"><h3 class="section-title mb-0">Lifts Progress & 1RM Gains</h3><div class="flex gap-2"><span class="badge" style="background:var(--accent-dim); color:var(--accent)">Epley Formula</span><span class="badge" style="background:rgba(0,230,118,0.1); color:var(--green)">Auto-updates</span></div></div><table class="lifts-table"><thead><tr><th style="min-width:180px">Lift</th><th>Wk 1</th><th>Wk 4</th><th>Wk 8</th><th>Wk 12</th><th>Latest</th><th>Change</th><th>% Change</th></tr></thead><tbody id="liftsTableBody"></tbody></table></section>

<section class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-5">
<div class="card" style="animation-delay:0.41s"><h3 class="section-title">Muscle Group Volume</h3><div class="chart-wrap flex items-center justify-center" style="height:200px"><canvas id="muscleChart"></canvas><div class="doughnut-center"><div class="font-mono text-2xl font-bold" style="color:var(--text)" id="muscleTotalSets">0</div><div class="text-[10px]" style="color:var(--text-muted)">Total Sets</div></div></div><div id="muscleLegend" class="mt-3 grid grid-cols-2 gap-x-3 gap-y-1.5"></div></div>
<div class="card" style="animation-delay:0.44s"><h3 class="section-title">Personal Records</h3><div id="prList"></div></div>
<div class="card" style="animation-delay:0.47s"><h3 class="section-title">Program Stats</h3><div id="statsList"></div></div>
</section>

<section class="card mb-5" style="animation-delay:0.5s; padding:1rem 1.25rem; overflow-x:auto"><h3 class="section-title">Weekly Breakdown</h3><table class="lifts-table"><thead><tr><th>Week</th><th>Day 1</th><th>Day 2</th><th>Day 3</th><th>Day 4</th><th>Day 5</th><th>Day 6</th><th>Total Sets</th><th>Avg RPE</th><th>Volume (kg)</th></tr></thead><tbody id="weeklyBreakdownBody"></tbody></table></section>

<footer class="text-center py-4" style="border-top:1px solid var(--border)"><p class="text-[11px]" style="color:var(--text-muted)">Gym - 12 Weeks Tracker July 2026 &bull; Dashboard &bull; Data from Google Sheets</p></footer>
</div>
</main>

<script>
// ==========================================
// CONFIGURATION - PASTE YOUR URL HERE
// ==========================================
const API_URL = https://script.google.com/macros/s/AKfycbxrZiT71jZBAKHkQGqLh8ETOYynTZhinqCMJAS-jBexlU_VZEin6HdLiaKRtq8xgoED3A/exec;

// Check if URL is configured
const isConfigured = API_URL && !API_URL.includes('PASTE_YOUR') && API_URL.includes('script.google.com');

let workoutData = null;
let charts = {};

function calc1RM(weight, reps) { const w = parseFloat(weight); const r = parseFloat(reps); if (!w || !r || w <= 0 || r <= 0) return 0; return w * (1 + r / 30); }
function getPhase(week) { if (week <= 4) return { num: 1, name: "Hypertrophy", color: "var(--accent)" }; if (week <= 8) return { num: 2, name: "Strength", color: "var(--cyan)" }; return { num: 3, name: "Peaking", color: "var(--lavender)" }; }

async function fetchData() {
    if (!isConfigured) {
        document.getElementById('loadingOverlay').classList.add('hidden');
        document.getElementById('dashboardContent').classList.add('hidden');
        document.getElementById('setupState').classList.remove('hidden');
        return null;
    }

    try {
        const response = await fetch(API_URL);
        if (!response.ok) throw new Error('HTTP ' + response.status);
        const data = await response.json();
        return processData(data);
    } catch (error) {
        console.error('Error fetching data:', error);
        document.getElementById('loadingOverlay').classList.add('hidden');
        document.getElementById('errorMessage').textContent = error.message + '. Check that your Google Apps Script is deployed and the sheet is shared.';
        document.getElementById('errorState').classList.remove('hidden');
        document.getElementById('dashboardContent').classList.add('hidden');
        return null;
    }
}

function processData(rawData) {
    const sessions = rawData.sessions || [];
    const metrics = {
        totalSessions: sessions.length, totalPossible: 72, totalSets: 0, totalTonnage: 0, totalRPE: 0, rpeCount: 0,
        best1RM: { value: 0, lift: '', week: 0 },
        weeklyData: Array(12).fill(null).map(() => ({ sets: [0,0,0,0,0,0], volume: [0,0,0,0,0,0], rpe: [], completed: [false,false,false,false,false,false] })),
        liftProgress: {}, muscleGroups: {}, personalRecords: []
    };

    sessions.forEach(session => {
        const w = session.week - 1; const d = session.day - 1;
        if (w < 0 || w >= 12 || d < 0 || d >= 6) return;
        let sessionSets = 0, sessionVolume = 0, sessionRPE = [];
        session.exercises.forEach(ex => {
            ex.sets.forEach((reps, i) => {
                const weight = ex.weights[i] || 0; const rpe = ex.rpe[i] || 0;
                sessionSets++; sessionVolume += weight * reps;
                if (rpe > 0) { sessionRPE.push(rpe); metrics.totalRPE += rpe; metrics.rpeCount++; }
                const cat = ex.category || 'Other';
                if (!metrics.muscleGroups[cat]) metrics.muscleGroups[cat] = 0;
                metrics.muscleGroups[cat]++;
                if (ex.keyLift) {
                    const rm = calc1RM(weight, reps);
                    if (rm > metrics.best1RM.value) metrics.best1RM = { value: rm, lift: ex.name, week: session.week };
                    if (!metrics.liftProgress[ex.name]) metrics.liftProgress[ex.name] = Array(12).fill(null);
                    const currentBest = metrics.liftProgress[ex.name][w];
                    if (!currentBest || rm > currentBest) metrics.liftProgress[ex.name][w] = rm;
                }
            });
        });
        metrics.weeklyData[w].sets[d] = sessionSets; metrics.weeklyData[w].volume[d] = sessionVolume;
        metrics.weeklyData[w].rpe.push(...sessionRPE); metrics.weeklyData[w].completed[d] = true;
        metrics.totalSets += sessionSets; metrics.totalTonnage += sessionVolume;
    });

    Object.entries(metrics.liftProgress).forEach(([lift, weeks]) => {
        let best = 0, bestWeek = 0;
        weeks.forEach((rm, w) => { if (rm && rm > best) { best = rm; bestWeek = w + 1; } });
        if (best > 0) metrics.personalRecords.push({ lift, value: best, week: bestWeek });
    });

    return { raw: rawData, metrics };
}

function renderDashboard(data) {
    if (!data) return;
    const { raw, metrics } = data; const currentWeek = raw.metadata?.currentWeek || 1; const phase = getPhase(currentWeek);
    document.getElementById('currentDate').textContent = new Date().toLocaleDateString('en-US', { month: 'long', year: 'numeric' });
    document.getElementById('currentPhase').textContent = `Phase ${phase.num} - ${phase.name}`;
    document.getElementById('weekIndicator').textContent = `Week ${currentWeek} of 12`;
    const markerPos = ((currentWeek - 1) / 11) * 100;
    document.getElementById('phaseMarker').style.left = `${Math.max(2, Math.min(98, markerPos))}%`;
    document.getElementById('metricSessions').textContent = metrics.totalSessions;
    document.getElementById('metricTotalSessions').textContent = metrics.totalPossible;
    document.getElementById('sessionsProgress').style.width = `${(metrics.totalSessions / metrics.totalPossible * 100)}%`;
    const adherence = metrics.totalPossible > 0 ? (metrics.totalSessions / metrics.totalPossible * 100) : 0;
    document.getElementById('metricAdherence').textContent = `${adherence.toFixed(1)}%`;
    const circumference = 2 * Math.PI * 28;
    const dashArray = (adherence / 100) * circumference;
    document.getElementById('adherenceRing').setAttribute('stroke-dasharray', `${dashArray} ${circumference}`);
    const avgRPE = metrics.rpeCount > 0 ? (metrics.totalRPE / metrics.rpeCount).toFixed(1) : null;
    document.getElementById('metricRPE').textContent = avgRPE || '-';
    document.getElementById('metricRPE').style.color = avgRPE ? 'var(--text)' : 'var(--text-muted)';
    document.getElementById('metricTonnage').textContent = `${Math.round(metrics.totalTonnage).toLocaleString()} kg`;
    if (metrics.best1RM.value > 0) { document.getElementById('metricBest1RM').textContent = `${metrics.best1RM.value.toFixed(1)} kg`; document.getElementById('metricBest1RMLabel').textContent = `${metrics.best1RM.lift} - Week ${metrics.best1RM.week}`; }
    document.getElementById('metricSets').textContent = metrics.totalSets;
    renderORMChart(metrics.liftProgress, currentWeek);
    renderVolumeChart(metrics.weeklyData);
    renderRPEChart(metrics.weeklyData);
    renderMuscleChart(metrics.muscleGroups, metrics.totalSets);
    renderSessionMatrix(metrics.weeklyData, currentWeek);
    renderLiftsTable(metrics.liftProgress);
    renderWeeklyBreakdown(metrics.weeklyData);
    renderPRs(metrics.personalRecords);
    renderStats(metrics, currentWeek);
    document.getElementById('loadingOverlay').classList.add('hidden');
    document.getElementById('dataSource').textContent = 'Live from Sheets';
}

function renderORMChart(liftProgress, currentWeek) {
    const ctx = document.getElementById('ormChart'); if (charts.orm) charts.orm.destroy();
    const colors = { 'Zercher Squat': '#c8ff00', 'Romanian Deadlift': '#00e5ff', 'Leg Press': '#ff6b6b', 'Leg Curl': '#ffab00', 'Standing Calf Raise': '#b388ff', 'Incline DB Press': '#ff80ab', 'Trap Bar Deadlift': '#69f0ae', 'Flat DB Press': '#82b1ff' };
    const datasets = Object.entries(liftProgress).map(([name, data]) => ({ label: name, data: data.map(v => v || null), borderColor: colors[name] || '#a0a0b8', backgroundColor: (colors[name] || '#a0a0b8') + '15', fill: true, tension: 0.35, pointRadius: data.map(v => v !== null ? 5 : 0), pointHoverRadius: 7, pointBackgroundColor: colors[name] || '#a0a0b8', pointBorderColor: '#13131c', pointBorderWidth: 2, borderWidth: 2.5, spanGaps: false }));
    document.getElementById('ormLegend').innerHTML = datasets.map(d => `<span class="flex items-center gap-1.5 text-[10px]" style="color:${d.borderColor}"><span class="inline-block w-2.5 h-2.5 rounded-full" style="background:${d.borderColor}"></span>${d.label}</span>`).join('');
    charts.orm = new Chart(ctx, { type: 'line', data: { labels: Array.from({length: 12}, (_, i) => `Wk ${i+1}`), datasets }, options: { responsive: true, maintainAspectRatio: false, interaction: { mode: 'index', intersect: false }, scales: { x: { grid: { color: 'rgba(42, 42, 63, 0.4)', drawBorder: false }, ticks: { font: { size: 10 }, color: '#5a5a72' } }, y: { grid: { color: 'rgba(42, 42, 63, 0.4)', drawBorder: false }, ticks: { font: { size: 10 }, color: '#5a5a72', callback: v => v + ' kg' }, beginAtZero: true } }, plugins: { legend: { display: false }, tooltip: { backgroundColor: '#1a1a26', titleColor: '#eaeaf0', bodyColor: '#a0a0b8', borderColor: '#2d2d3d', borderWidth: 1, padding: 10, cornerRadius: 8, callbacks: { label: ctx => ctx.parsed.y !== null ? `${ctx.dataset.label}: ${ctx.parsed.y.toFixed(1)} kg` : null } } }, animation: { duration: 1200, easing: 'easeOutQuart' } } });
}

function renderVolumeChart(weeklyData) {
    const ctx = document.getElementById('volumeChart'); if (charts.volume) charts.volume.destroy();
    const dayColors = ['#c8ff00', '#00e5ff', '#ff6b6b', '#ffab00', '#b388ff', '#69f0ae'];
    const dayNames = ['Lower A', 'Upper A', 'Condition', 'Zone 2', 'Lower B', 'Upper B'];
    const datasets = dayNames.map((name, i) => ({ label: name, data: weeklyData.map(w => w.sets[i]), backgroundColor: dayColors[i] + 'cc', hoverBackgroundColor: dayColors[i], borderRadius: 3, borderSkipped: false }));
    charts.volume = new Chart(ctx, { type: 'bar', data: { labels: Array.from({length: 12}, (_, i) => `Wk ${i+1}`), datasets }, options: { responsive: true, maintainAspectRatio: false, scales: { x: { stacked: true, grid: { display: false }, ticks: { font: { size: 10 }, color: '#5a5a72' } }, y: { stacked: true, grid: { color: 'rgba(42, 42, 63, 0.4)', drawBorder: false }, ticks: { font: { size: 10 }, color: '#5a5a72', stepSize: 10 }, beginAtZero: true } }, plugins: { legend: { display: false }, tooltip: { backgroundColor: '#1a1a26', titleColor: '#eaeaf0', bodyColor: '#a0a0b8', borderColor: '#2d2d3d', borderWidth: 1, padding: 10, cornerRadius: 8, callbacks: { label: ctx => ctx.parsed.y > 0 ? `${ctx.dataset.label}: ${ctx.parsed.y} sets` : null } } }, animation: { duration: 1000, easing: 'easeOutQuart' } } });
}

function renderRPEChart(weeklyData) {
    const ctx = document.getElementById('rpeChart'); if (charts.rpe) charts.rpe.destroy();
    const avgRPEs = weeklyData.map(w => w.rpe.length === 0 ? null : w.rpe.reduce((a, b) => a + b, 0) / w.rpe.length);
    const hasData = avgRPEs.some(v => v !== null);
    document.getElementById('rpeEmptyState').style.display = hasData ? 'none' : 'flex';
    charts.rpe = new Chart(ctx, { type: 'line', data: { labels: Array.from({length: 12}, (_, i) => `Wk ${i+1}`), datasets: [{ label: 'Avg RPE', data: avgRPEs, borderColor: '#ff6b6b', backgroundColor: '#ff6b6b15', fill: true, tension: 0.35, borderWidth: 2.5, pointRadius: avgRPEs.map(v => v !== null ? 5 : 0), pointBackgroundColor: '#ff6b6b', spanGaps: false }] }, options: { responsive: true, maintainAspectRatio: false, scales: { x: { grid: { color: 'rgba(42, 42, 63, 0.4)', drawBorder: false }, ticks: { font: { size: 10 }, color: '#5a5a72' } }, y: { grid: { color: 'rgba(42, 42, 63, 0.4)', drawBorder: false }, ticks: { font: { size: 10 }, color: '#5a5a72', stepSize: 1 }, min: 1, max: 10 } }, plugins: { legend: { display: false }, tooltip: { backgroundColor: '#1a1a26', titleColor: '#eaeaf0', bodyColor: '#a0a0b8', borderColor: '#2d2d3d', borderWidth: 1, padding: 10, cornerRadius: 8 } } } });
}

function renderMuscleChart(muscleGroups, totalSets) {
    const ctx = document.getElementById('muscleChart'); if (charts.muscle) charts.muscle.destroy();
    const muscleColors = { 'Quad': '#c8ff00', 'Hinge': '#00e5ff', 'Hamstring': '#ffab00', 'Glutes': '#ff80ab', 'Calves': '#b388ff', 'Chest': '#ff6b6b', 'Horizontal Pull': '#69f0ae', 'Vertical Pull': '#82b1ff', 'Shoulder': '#ff9100', 'Side Delt': '#00e676', 'Rear Delt': '#a0a0b8', 'Biceps': '#ff4d6a', 'Triceps': '#00e5ff', 'Core': '#ff6b6b', 'Conditioning': '#ffab00', 'Loaded Carry': '#b388ff', 'Shoulder Health': '#a0a0b8' };
    const groups = Object.entries(muscleGroups).sort((a, b) => b[1] - a[1]).slice(0, 8);
    document.getElementById('muscleTotalSets').textContent = totalSets;
    charts.muscle = new Chart(ctx, { type: 'doughnut', data: { labels: groups.map(g => g[0]), datasets: [{ data: groups.map(g => g[1]), backgroundColor: groups.map(g => muscleColors[g[0]] || '#a0a0b8' + 'cc'), hoverBackgroundColor: groups.map(g => muscleColors[g[0]] || '#a0a0b8'), borderColor: '#13131c', borderWidth: 3 }] }, options: { responsive: true, maintainAspectRatio: false, cutout: '72%', plugins: { legend: { display: false }, tooltip: { backgroundColor: '#1a1a26', titleColor: '#eaeaf0', bodyColor: '#a0a0b8', borderColor: '#2d2d3d', borderWidth: 1, padding: 10, cornerRadius: 8, callbacks: { label: ctx => `${ctx.label}: ${ctx.parsed} sets (${Math.round(ctx.parsed / totalSets * 100)}%)` } } }, animation: { animateRotate: true, duration: 1200 } } });
    document.getElementById('muscleLegend').innerHTML = groups.map(g => { const pct = Math.round(g[1] / totalSets * 100); const color = muscleColors[g[0]] || '#a0a0b8'; return `<div class="flex items-center gap-2"><span class="w-2 h-2 rounded-full flex-shrink-0" style="background:${color}"></span><span class="text-[11px]" style="color:var(--text-secondary)">${g[0]}</span><span class="text-[11px] font-mono ml-auto" style="color:var(--text-muted)">${g[1]} <span style="opacity:0.5">(${pct}%)</span></span></div>`; }).join('');
}

function renderSessionMatrix(weeklyData, currentWeek) {
    const matrixEl = document.getElementById('sessionMatrix'); matrixEl.innerHTML = '';
    const dayNames = ['Lower A', 'Upper A', 'Condition', 'Zone 2', 'Lower B', 'Upper B'];
    for (let w = 0; w < 12; w++) {
        const label = document.createElement('div'); label.className = 'text-[9px] font-mono flex items-center justify-end pr-1'; label.style.color = 'var(--text-muted)'; label.textContent = `${w + 1}`; matrixEl.appendChild(label);
        for (let d = 0; d < 6; d++) {
            const cell = document.createElement('div'); cell.className = 'matrix-cell';
            const isCompleted = weeklyData[w]?.completed?.[d];
            const isCurrentWeek = w === currentWeek - 1; const isPast = w < currentWeek - 1;
            if (isCompleted) { cell.classList.add('completed'); cell.setAttribute('data-tip', `${dayNames[d]} - Week ${w+1}: Completed`); }
            else if (isCurrentWeek) { cell.classList.add('current'); cell.setAttribute('data-tip', `${dayNames[d]} - Week ${w+1}: Upcoming`); }
            else if (isPast) { cell.classList.add('missed'); cell.setAttribute('data-tip', `${dayNames[d]} - Week ${w+1}: Missed`); }
            else { cell.classList.add('future'); cell.setAttribute('data-tip', `${dayNames[d]} - Week ${w+1}`); }
            matrixEl.appendChild(cell);
        }
    }
}

function renderLiftsTable(liftProgress) {
    const tbody = document.getElementById('liftsTableBody'); tbody.innerHTML = '';
    const colors = { 'Zercher Squat': '#c8ff00', 'Romanian Deadlift': '#00e5ff', 'Leg Press': '#ff6b6b', 'Leg Curl': '#ffab00', 'Standing Calf Raise': '#b388ff', 'Incline DB Press': '#ff80ab', 'Trap Bar Deadlift': '#69f0ae', 'Flat DB Press': '#82b1ff' };
    Object.entries(liftProgress).forEach(([name, weeks]) => {
        const wk1 = weeks[0]; const wk4 = weeks[3]; const wk8 = weeks[7]; const wk12 = weeks[11];
        const latest = weeks.slice().reverse().find(v => v !== null) ?? null;
        const first = wk1; const change = latest !== null && first !== null ? latest - first : null;
        const pctChange = first !== null && change !== null ? (change / first * 100) : null;
        const changeColor = change === null ? 'var(--text-muted)' : change > 0 ? 'var(--green)' : change < 0 ? 'var(--red)' : 'var(--text-muted)';
        const fmt = v => v !== null ? v.toFixed(1) : '<span style="color:var(--text-muted)">-</span>';
        const row = document.createElement('tr');
        row.innerHTML = `<td style="color:var(--text); font-weight:500"><span class="inline-block w-2 h-2 rounded-full mr-2" style="background:${colors[name] || '#a0a0b8'}"></span>${name}</td><td class="font-mono text-sm">${fmt(wk1)}</td><td class="font-mono text-sm">${fmt(wk4)}</td><td class="font-mono text-sm">${fmt(wk8)}</td><td class="font-mono text-sm">${fmt(wk12)}</td><td class="font-mono text-sm" style="color:var(--text); font-weight:600">${fmt(latest)}</td><td class="font-mono text-sm" style="color:${changeColor}">${change !== null ? (change > 0 ? '+' : '') + change.toFixed(1) : '-'}</td><td class="font-mono text-sm" style="color:${changeColor}">${pctChange !== null ? pctChange.toFixed(1) + '%' : '-'}</td>`;
        tbody.appendChild(row);
    });
}

function renderWeeklyBreakdown(weeklyData) {
    const tbody = document.getElementById('weeklyBreakdownBody'); tbody.innerHTML = '';
    weeklyData.forEach((week, w) => {
        const totalSets = week.sets.reduce((a, b) => a + b, 0);
        const totalVolume = week.volume.reduce((a, b) => a + b, 0);
        const avgRPE = week.rpe.length > 0 ? (week.rpe.reduce((a, b) => a + b, 0) / week.rpe.length).toFixed(1) : '-';
        const row = document.createElement('tr');
        row.innerHTML = `<td class="font-mono text-sm font-semibold" style="color:var(--text)">Wk ${w+1}</td>${week.sets.map((sets, d) => `<td class="font-mono text-sm" style="color:${sets > 0 ? 'var(--accent)' : 'var(--text-muted)'}; opacity:${sets > 0 ? 1 : 0.4}">${sets > 0 ? sets : '-'}</td>`).join('')}<td class="font-mono text-sm font-semibold" style="color:var(--text)">${totalSets}</td><td class="font-mono text-sm" style="color:var(--text-secondary)">${avgRPE}</td><td class="font-mono text-sm" style="color:var(--cyan)">${totalVolume > 0 ? totalVolume.toLocaleString() : '-'}</td>`;
        tbody.appendChild(row);
    });
}

function renderPRs(personalRecords) {
    const prListEl = document.getElementById('prList');
    if (personalRecords.length === 0) { prListEl.innerHTML = '<div class="text-sm text-center py-6" style="color:var(--text-muted)">Complete a session to set PRs</div>'; return; }
    prListEl.innerHTML = personalRecords.sort((a, b) => b.value - a.value).map(pr => `<div class="pr-item"><div><div class="text-sm font-medium" style="color:var(--text)">${pr.lift}</div><div class="text-[10px]" style="color:var(--text-muted)">Set in Week ${pr.week}</div></div><div class="text-right"><div class="font-mono text-sm font-bold" style="color:var(--accent)">${pr.value.toFixed(1)}</div><div class="text-[10px]" style="color:var(--text-muted)">kg est. 1RM</div></div></div>`).join('');
}

function calculateStreak(weeklyData, currentWeek) {
    let streak = 0;
    for (let w = currentWeek - 1; w >= 0; w--) {
        const hasAny = weeklyData[w]?.completed?.some(Boolean);
        if (hasAny) streak++; else break;
    }
    return streak;
}

function renderStats(metrics, currentWeek) {
    const statsEl = document.getElementById('statsList');
    const totalSessions = metrics.totalSessions;
    const avgSetsPerSession = totalSessions > 0 ? (metrics.totalSets / totalSessions).toFixed(1) : '0';
    const avgTonnagePerSession = totalSessions > 0 ? Math.round(metrics.totalTonnage / totalSessions).toLocaleString() : '0';
    const currentStreak = calculateStreak(metrics.weeklyData, currentWeek);
    const stats = [
        { label: 'Avg Sets / Session', value: avgSetsPerSession, icon: 'fa-layer-group' },
        { label: 'Avg Tonnage / Session', value: `${avgTonnagePerSession} kg`, icon: 'fa-weight-hanging' },
        { label: 'Current Streak', value: `${currentStreak} week${currentStreak !== 1 ? 's' : ''}`, icon: 'fa-bolt' },
        { label: 'Sessions This Week', value: `${metrics.weeklyData[currentWeek-1]?.completed?.filter(Boolean)?.length || 0} / 6`, icon: 'fa-calendar-check' },
        { label: 'Total Exercises', value: Object.keys(metrics.liftProgress).length, icon: 'fa-dumbbell' },
        { label: 'Muscle Groups Hit', value: Object.keys(metrics.muscleGroups).length, icon: 'fa-heart-pulse' },
    ];
    statsEl.innerHTML = stats.map(s => `<div class="stat-row"><div class="flex items-center gap-2"><i class="fa-solid ${s.icon} text-[11px]" style="color:var(--text-muted); width:16px; text-align:center"></i><span class="text-xs" style="color:var(--text-secondary)">${s.label}</span></div><span class="font-mono text-xs font-semibold" style="color:var(--text)">${s.value}</span></div>`).join('');
}

function refreshData() { 
    document.getElementById('loadingOverlay').classList.remove('hidden');
    document.getElementById('errorState').classList.add('hidden');
    fetchData().then(data => { if (data) renderDashboard(data); });
}

// Initialize
fetchData().then(data => { if (data) renderDashboard(data); });
</script>
</body>
</html>
