<script>
  let frequency = 10;
  let sliderMax = 120;
  let fgColor = '#000000';
  let bgColor = '#ffffff';
  let isPlaying = false;
  let flickerInterval = null;
  let isOn = false;
  let theme = 'dark';

  import { onMount, onDestroy } from 'svelte';

  onMount(() => {
    const stored = localStorage.getItem('theme');
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    theme = stored || (prefersDark ? 'dark' : 'light');
    document.documentElement.setAttribute('data-theme', theme);
  });

  function toggleTheme() {
    theme = theme === 'dark' ? 'light' : 'dark';
    document.documentElement.setAttribute('data-theme', theme);
    localStorage.setItem('theme', theme);
  }

  function startFlicker() {
    isPlaying = true;
    isOn = true;
    restartFlickerInterval();
  }

  function restartFlickerInterval() {
    if (!Number.isFinite(frequency) || frequency <= 0) {
      return;
    }
    if (flickerInterval) {
      clearInterval(flickerInterval);
    }
    flickerInterval = setInterval(toggleFlicker, 1000 / (frequency * 2));
  }

  function toggleFlicker() {
    isOn = !isOn;
  }

  function stopFlicker() {
    isPlaying = false;
    isOn = false;
    if (flickerInterval) {
      clearInterval(flickerInterval);
      flickerInterval = null;
    }
  }

  function togglePlay() {
    if (isPlaying) stopFlicker();
    else startFlicker();
  }

  onDestroy(() => {
    stopFlicker();
  });

  $: flickerStyle = `background:${isOn ? fgColor : bgColor};transition:none;`;

  $: if (isPlaying && frequency > 0) {
    restartFlickerInterval();
  }

  $: if (frequency > sliderMax) {
    sliderMax = Math.ceil(frequency);
  }

  $: freqDisplay = frequency < 1 ? frequency.toFixed(1) : Math.round(frequency);

</script>

<div class="app-container">
  <!-- Header -->
  <div class="header">
    <h1>SCREEN FLICKER</h1>
    <div class="header-right">
      <button class="theme-btn" on:click={toggleTheme} aria-label="Toggle theme">
        {#if theme === 'dark'}
          <svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <circle cx="12" cy="12" r="4"></circle>
            <path d="M12 2v2M12 20v2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M2 12h2M20 12h2M4.93 19.07l1.41-1.41M17.66 6.34l1.41-1.41"></path>
          </svg>
        {:else}
          <svg viewBox="0 0 24 24" width="18" height="18" fill="currentColor" aria-hidden="true">
            <path d="M20.354 15.354A9 9 0 018.646 3.646 9 9 0 1012 21a8.96 8.96 0 008.354-5.646z"></path>
          </svg>
        {/if}
      </button>
    </div>
  </div>

  <!-- Controls Panel -->
  <div class="controls-panel">
    <!-- Frequency -->
    <div class="control-group">
      <label class="control-label">Flicker Frequency</label>
      <div class="slider-row">
        <input type="range" min="0.5" max={sliderMax} step="0.5" bind:value={frequency} />
        <span class="value-badge">{freqDisplay} Hz</span>
      </div>
      <div class="freq-input-row">
        <label for="frequency-input">Frequency (Hz)</label>
        <input id="frequency-input" type="number" min="0.1" step="0.1" bind:value={frequency} />
      </div>
      <div class="freq-input-row">
        <label for="slider-max-input">Slider max (Hz)</label>
        <input id="slider-max-input" type="number" min="1" step="1" bind:value={sliderMax} />
      </div>
      <div class="freq-markers">
        <span>0.5</span><span>{Math.round(sliderMax * 0.25)}</span><span>{Math.round(sliderMax * 0.5)}</span><span>{Math.round(sliderMax * 0.75)}</span><span>{Math.round(sliderMax)}</span>
      </div>
    </div>

    <!-- Colors -->
    <div class="control-group">
      <label class="control-label">Colors</label>
      <div class="color-row">
        <div class="color-picker">
          <label>Foreground</label>
          <input type="color" bind:value={fgColor} />
        </div>
        <div class="color-swap">
          <button class="swap-btn" on:click={() => { const tmp = fgColor; fgColor = bgColor; bgColor = tmp; }} aria-label="Swap colors">
            <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M7 16l-4-4 4-4"/><path d="M17 8l4 4-4 4"/>
              <path d="M3 12h18"/>
            </svg>
          </button>
        </div>
        <div class="color-picker">
          <label>Background</label>
          <input type="color" bind:value={bgColor} />
        </div>
      </div>
    </div>

    <!-- Action Buttons -->
    <div class="action-row">
      <button class="play-btn" class:playing={isPlaying} on:click={togglePlay}>
        {isPlaying ? '■ STOP' : '▶ START'}
      </button>
    </div>

    <!-- Preview Area -->
    <div class="control-group preview-group">
      <label class="control-label">Preview</label>
      <p class="preview-help">Drag the bottom-right corner to resize the preview.</p>
      <div class="preview-resizer">
        <div class="preview-area" style={flickerStyle}></div>
      </div>
    </div>
  </div>
</div>

<style>
  .app-container {
    max-width: 540px;
    margin: 0 auto;
    padding: 0 20px 40px;
  }

  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 16px 0;
    border-bottom: 1px solid var(--border-color);
    margin-bottom: 32px;
  }

  h1 {
    font-size: 1.1rem;
    font-weight: 700;
    letter-spacing: 0.05em;
    color: var(--text-primary);
  }

  .theme-btn {
    background: none;
    border: 1px solid var(--border-color);
    border-radius: 999px;
    width: 32px;
    height: 32px;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--text-primary);
    cursor: pointer;
    margin-left: auto;
    line-height: 0;
  }
  .theme-btn:hover { border-color: var(--accent); }

  .theme-btn svg {
    display: block;
    width: 18px;
    height: 18px;
    flex-shrink: 0;
    pointer-events: none;
  }

  .controls-panel {
    display: flex;
    flex-direction: column;
    gap: 20px;
  }

  .control-group {
    background: var(--surface-1);
    border: 1px solid var(--border-color);
    border-radius: 12px;
    padding: 20px;
  }

  .control-label {
    font-size: 0.8rem;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--text-secondary);
    margin-bottom: 12px;
  }

  .slider-row {
    display: flex;
    align-items: center;
    gap: 12px;
  }

  .slider-row input[type="range"] { flex: 1; }

  .value-badge {
    font-size: 1rem;
    font-weight: 700;
    color: var(--accent);
    min-width: 52px;
    text-align: right;
    font-variant-numeric: tabular-nums;
  }

  .freq-markers {
    display: flex;
    justify-content: space-between;
    margin-top: 6px;
    font-size: 0.65rem;
    color: var(--text-secondary);
    padding: 0 2px;
  }

  .freq-input-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 10px;
    margin-top: 8px;
    font-size: 0.75rem;
    color: var(--text-secondary);
  }

  .freq-input-row input {
    width: 120px;
    background: var(--surface-2);
    color: var(--text-primary);
    border: 1px solid var(--border-color);
    border-radius: 8px;
    padding: 6px 8px;
    font-size: 0.85rem;
  }

  .color-row {
    display: flex;
    align-items: center;
    gap: 12px;
  }

  .color-picker {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 6px;
  }

  .color-picker label {
    font-size: 0.75rem;
    color: var(--text-secondary);
  }

  .color-swap {
    display: flex;
    align-items: center;
  }

  .swap-btn {
    background: var(--surface-2);
    border: 1px solid var(--border-color);
    border-radius: 8px;
    padding: 8px;
    color: var(--text-primary);
    cursor: pointer;
    transition: all 0.15s;
  }
  .swap-btn:hover { border-color: var(--accent); color: var(--accent); }

  .action-row {
    display: flex;
    gap: 10px;
  }

  .play-btn {
    flex: 1;
    padding: 14px;
    font-size: 1rem;
    font-weight: 700;
    letter-spacing: 0.05em;
    border-radius: 12px;
    background: var(--accent);
    color: white;
    border: none;
    transition: all 0.2s;
  }

  .play-btn:hover { background: var(--accent-hover); }

  .play-btn.playing {
    background: var(--danger);
  }

  .play-btn.playing:hover {
    background: #c82333;
  }

  .preview-group {
    margin-bottom: 0;
  }

  .preview-help {
    font-size: 0.75rem;
    color: var(--text-secondary);
    margin-bottom: 10px;
  }

  .preview-resizer {
    width: 100%;
    min-height: 120px;
    max-height: 60vh;
    resize: both;
    overflow: auto;
    border: 1px dashed var(--border-color);
    border-radius: 8px;
  }

  .preview-area {
    width: 100%;
    height: 100%;
    min-height: 120px;
    border-radius: 8px;
    border: 1px solid transparent;
  }

  @media (max-width: 480px) {
    .action-row { flex-direction: column; }
    .play-btn { width: 100%; }
  }
</style>
