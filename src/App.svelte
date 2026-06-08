<script>
  let frequency = 10;
  let fgColor = '#000000';
  let bgColor = '#ffffff';
  let isPlaying = false;
  let flickerInterval = null;
  let isOn = false;
  let isFullscreen = false;
  let theme = 'dark';
  let fullscreenContainer;

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
    toggleFlicker();
    flickerInterval = setInterval(toggleFlicker, 1000 / (frequency * 2));
  }

  function toggleFlicker() {
    isOn = !isOn;
  }

  function stopFlicker() {
    isPlaying = false;
    if (flickerInterval) {
      clearInterval(flickerInterval);
      flickerInterval = null;
    }
  }

  function togglePlay() {
    if (isPlaying) stopFlicker();
    else startFlicker();
  }

  function toggleFullscreen() {
    if (!document.fullscreenElement) {
      fullscreenContainer.requestFullscreen();
      isFullscreen = true;
    } else {
      document.exitFullscreen();
      isFullscreen = false;
    }
  }

  function onFsChange() {
    isFullscreen = !!document.fullscreenElement;
    if (!isFullscreen && !isPlaying) {
      // Reset view when exiting fullscreen
    }
  }

  onDestroy(() => {
    stopFlicker();
    document.removeEventListener('fullscreenchange', onFsChange);
  });

  $: flickerStyle = {
    background: isOn ? fgColor : bgColor,
    transition: 'none'
  };

  $: freqDisplay = frequency < 1 ? frequency.toFixed(1) : Math.round(frequency);

  import { onMount as onM } from 'svelte';
  // Re-attach listener on mount
  import { afterUpdate } from 'svelte';
  afterUpdate(() => {
    if (fullscreenContainer) {
      document.addEventListener('fullscreenchange', onFsChange);
    }
  });
</script>

<div class="app-container">
  <!-- Header -->
  <div class="header">
    <div class="header-left">
      <a href="/" class="home-link">← Home</a>
    </div>
    <div class="header-center">
      <h1>SCREEN FLICKER</h1>
    </div>
    <div class="header-right">
      <button class="theme-btn" on:click={toggleTheme} aria-label="Toggle theme">
        {#if theme === 'dark'}
          <svg viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="12" cy="12" r="4"></circle>
            <path d="M12 2v2M12 20v2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M2 12h2M20 12h2M4.93 19.07l1.41-1.41M17.66 6.34l1.41-1.41"></path>
          </svg>
        {:else}
          <svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor">
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
        <input type="range" min="0.5" max="60" step="0.5" bind:value={frequency} />
        <span class="value-badge">{freqDisplay} Hz</span>
      </div>
      <div class="freq-markers">
        <span>0.5</span><span>10</span><span>20</span><span>30</span><span>60</span>
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
      <button class="fullscreen-btn" on:click={toggleFullscreen}>
        {isFullscreen ? '⊞ Exit Fullscreen' : '⊞ Fullscreen'}
      </button>
    </div>

    <!-- Preview Area -->
    <div class="control-group preview-group">
      <label class="control-label">Preview</label>
      <div class="preview-area" style={flickerStyle}></div>
    </div>
  </div>
</div>

<!-- Fullscreen Flicker Overlay -->
<div class="flicker-overlay" style={flickerStyle} class:visible={isPlaying} bind:this={fullscreenContainer}>
  {#if isPlaying && !isFullscreen}
    <div class="overlay-hint">Press fullscreen for best effect</div>
  {/if}
  {#if isPlaying && isFullscreen}
    <div class="overlay-close" on:click={stopFlicker}>✕ Stop</div>
  {/if}
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

  .header-left, .header-right { flex: 1; }
  .header-center { text-align: center; }

  .home-link {
    color: var(--accent);
    text-decoration: none;
    font-size: 0.85rem;
  }
  .home-link:hover { color: var(--accent-hover); }

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
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--text-primary);
    cursor: pointer;
    margin-left: auto;
  }
  .theme-btn:hover { border-color: var(--accent); }

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

  .fullscreen-btn {
    padding: 14px 18px;
    font-size: 0.85rem;
    font-weight: 600;
    border-radius: 12px;
    background: var(--surface-2);
    color: var(--text-primary);
    border: 1px solid var(--border-color);
    transition: all 0.2s;
    white-space: nowrap;
  }

  .fullscreen-btn:hover {
    border-color: var(--accent);
    color: var(--accent);
  }

  .preview-group {
    margin-bottom: 0;
  }

  .preview-area {
    width: 100%;
    height: 120px;
    border-radius: 8px;
    border: 1px solid var(--border-color);
  }

  .flicker-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    z-index: 9999;
    display: none;
    cursor: pointer;
  }

  .flicker-overlay.visible {
    display: block;
  }

  .overlay-hint {
    position: absolute;
    bottom: 24px;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(0,0,0,0.6);
    color: white;
    padding: 8px 16px;
    border-radius: 8px;
    font-size: 0.8rem;
    pointer-events: none;
  }

  .overlay-close {
    position: absolute;
    top: 16px;
    right: 16px;
    background: rgba(0,0,0,0.6);
    color: white;
    padding: 8px 16px;
    border-radius: 8px;
    font-size: 0.9rem;
    font-weight: 600;
    cursor: pointer;
    z-index: 10000;
    border: none;
  }

  .overlay-close:hover {
    background: rgba(220,53,69,0.8);
  }

  @media (max-width: 480px) {
    .action-row { flex-direction: column; }
    .fullscreen-btn { width: 100%; }
  }
</style>
