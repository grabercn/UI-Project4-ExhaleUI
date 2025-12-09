<script>
  import { onMount, onDestroy } from 'svelte';
  import { scale, fade } from 'svelte/transition';

  // --- CONFIGURATION ---
  const techniques = {
    box: {
      label: "Box Breathing",
      desc: "Equal duration for all four phases. Best for emergency stress regulation and focus (Navy SEAL method).",
      sequence: [
        { text: "Inhale", time: 4000, scale: 1, glow: 1.5 },
        { text: "Hold", time: 4000, scale: 1, glow: 1.5 },
        { text: "Exhale", time: 4000, scale: 0.4, glow: 1 },
        { text: "Hold", time: 4000, scale: 0.4, glow: 1 }
      ]
    },
    relax: {
      label: "4-7-8 Relax",
      desc: "Extended exhale activates the parasympathetic nervous system. Best for sleep and anxiety.",
      sequence: [
        { text: "Inhale", time: 4000, scale: 1, glow: 1.5 },
        { text: "Hold", time: 7000, scale: 1, glow: 1.5 },
        { text: "Exhale", time: 8000, scale: 0.4, glow: 1 }
      ]
    },
    coherence: {
      label: "Coherence",
      desc: "Rhythmic 5.5s breathing. Best for heart-rate variability and emotional balance.",
      sequence: [
        { text: "Inhale", time: 5500, scale: 1, glow: 1.5 },
        { text: "Exhale", time: 5500, scale: 0.4, glow: 1 }
      ]
    },
    deep: {
      label: "Deep Calm",
      desc: "Slow, deep breaths with a long hold at the top. Great for tension headaches.",
      sequence: [
        { text: "Inhale", time: 5000, scale: 1, glow: 1.5 },
        { text: "Hold", time: 5000, scale: 1, glow: 1.5 },
        { text: "Exhale", time: 7000, scale: 0.4, glow: 1 }
      ]
    },
    awake: {
      label: "Awake (Energy)",
      desc: "Fast, energizing breathwork. Use this if you feel sluggish (Coffee replacement).",
      sequence: [
        { text: "Inhale", time: 3000, scale: 1, glow: 1.5 },
        { text: "Exhale", time: 2000, scale: 0.4, glow: 1 }
      ]
    }
  };

  // --- STATE ---
  let isRunning = false;
  let currentText = "Ready?";
  let circleScale = 0.4;
  let shadowIntensity = 1; // 1 = normal, 1.5 = glowing
  let transitionTime = 500;
  let selectedTechnique = "box";
  let showInfo = false;
  let timeoutId;
  let isFavorite = false;

  // --- LIFECYCLE ---
  onMount(() => {
    const fav = localStorage.getItem('exhale_favorite');
    if (fav && techniques[fav]) {
      selectedTechnique = fav;
      isFavorite = true;
    }
  });

  onDestroy(() => clearTimeout(timeoutId));

  // --- REACTIVITY ---
  $: {
    const saved = localStorage.getItem('exhale_favorite');
    isFavorite = (saved === selectedTechnique);
  }

  // --- LOGIC ---
  function toggleFavorite() {
    if (isFavorite) {
      localStorage.removeItem('exhale_favorite');
      isFavorite = false;
    } else {
      localStorage.setItem('exhale_favorite', selectedTechnique);
      isFavorite = true;
    }
  }

  function runCycle(techName, index = 0) {
    if (!isRunning) return;

    const sequence = techniques[techName].sequence;
    const phase = sequence[index];

    // apply the new state
    currentText = phase.text;
    transitionTime = phase.time;
    circleScale = phase.scale;
    shadowIntensity = phase.glow;
    
    if (navigator.vibrate) navigator.vibrate(50);

    timeoutId = setTimeout(() => {
      const nextIndex = (index + 1) % sequence.length;
      runCycle(techName, nextIndex);
    }, phase.time);
  }

  function toggle() {
    if (isRunning) {
      stop();
    } else {
      start();
    }
  }

  function start() {
    // first set running to true, removes the idle css class immediately
    isRunning = true;
    showInfo = false;

    // we have to wait 50ms before triggering the growth effect
    setTimeout(() => {
        runCycle(selectedTechnique);
    }, 50);
  }

  function stop() {
    isRunning = false;
    clearTimeout(timeoutId);
    currentText = "Ready?";
    // force reset to small size
    circleScale = 0.4;
    shadowIntensity = 1;
    // transition for the reset
    transitionTime = 600; 
  }
</script>

<div class="breathing-container">
  
  {#if showInfo}
    <div class="info-popup" transition:scale={{duration: 200}}>
      <h4>{techniques[selectedTechnique].label}</h4>
      <p>{techniques[selectedTechnique].desc}</p>
      <button class="close-info" on:click={() => showInfo = false}>Got it</button>
    </div>
  {/if}

  <div 
    class="visualizer-area" 
    on:click={toggle}
    role="button"
    tabindex="0"
    on:keydown={(e) => (e.key === 'Enter' || e.key === ' ') && toggle()}
    aria-label={isRunning ? "Stop breathing exercise" : "Start breathing exercise"}
  >
    <div class="circle-wrapper">
      <div class="circle-bg"></div>
      
      <div 
        class="breathing-circle"
        class:idle={!isRunning} 
        style="
          transform: scale({circleScale}); 
          transition-duration: {transitionTime}ms;
          box-shadow: 0 0 {25 * shadowIntensity}px rgba(255,255,255, {0.5 * shadowIntensity});
        ">
      </div>
      
      <div class="instruction-text">
        {currentText}
      </div>
    </div>
  </div>

  <div class="controls-area">
    <div class="selector-row">
      <select bind:value={selectedTechnique} disabled={isRunning} on:change={() => showInfo = false} on:click|stopPropagation>
        {#each Object.entries(techniques) as [key, tech]}
          <option value={key}>{tech.label}</option>
        {/each}
      </select>

      <button 
        class="icon-btn" 
        class:active={isFavorite}
        on:click|stopPropagation={toggleFavorite}
        aria-label="Set as favorite">
        {isFavorite ? '★' : '☆'}
      </button>

      <button 
        class="icon-btn info-btn" 
        on:click|stopPropagation={() => showInfo = !showInfo}
        aria-label="Technique Information">
        i
      </button>
    </div>

    <button class="btn-primary" on:click={toggle}>
      {isRunning ? 'Stop' : 'Start'}
    </button>
  </div>

</div>

<style>
  .breathing-container {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    position: relative;
  }

  .visualizer-area {
    flex: 1;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    outline: none;
    -webkit-tap-highlight-color: transparent;
  }

  .circle-wrapper {
    position: relative;
    width: 280px;
    height: 280px;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .circle-bg {
    position: absolute;
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background: #fff;
    opacity: 0.1;
    z-index: 0;
  }

  .breathing-circle {
    width: 100%;
    height: 100%;
    background: var(--circle-color);
    border-radius: 50%;
    position: absolute;
    z-index: 1;
    will-change: transform, box-shadow;
    transition-timing-function: cubic-bezier(0.25, 0.46, 0.45, 0.94);
  }

  .breathing-circle.idle {
    animation: idlePulse 3s infinite ease-in-out;
  }

  @keyframes idlePulse {
    0% { transform: scale(0.4); opacity: 0.9; }
    50% { transform: scale(0.43); opacity: 1; }
    100% { transform: scale(0.4); opacity: 0.9; }
  }

  .instruction-text {
    position: relative;
    z-index: 2;
    font-size: 1.6rem;
    font-weight: 700;
    color: var(--text-primary);
    text-shadow: 0 2px 10px rgba(255,255,255,0.5);
    pointer-events: none;
    user-select: none;
  }

  .controls-area {
    width: 100%;
    padding: 0 20px 20px 20px;
    display: flex;
    flex-direction: column;
    gap: 15px;
    z-index: 10;
  }

  .selector-row {
    display: flex;
    gap: 10px;
    height: 50px;
  }

  select {
    flex: 1;
    padding: 0 15px;
    border-radius: 12px;
    border: none;
    background: rgba(255,255,255,0.5);
    font-size: 1rem;
    color: var(--text-primary);
    font-family: inherit;
    font-weight: 600;
    box-shadow: 0 2px 10px rgba(0,0,0,0.05);
  }

  .icon-btn {
    width: 50px;
    height: 50px;
    border-radius: 12px;
    border: none;
    background: rgba(255,255,255,0.4);
    font-size: 1.2rem;
    color: var(--text-secondary);
    cursor: pointer;
    display: flex;
    justify-content: center;
    align-items: center;
    transition: all 0.2s;
    box-shadow: 0 2px 10px rgba(0,0,0,0.05);
  }
  .icon-btn:hover { background: rgba(255,255,255,0.6); }
  .icon-btn.active { color: #ecc94b; background: rgba(255,255,255,0.7); }
  .info-btn { font-family: serif; font-weight: bold; font-style: italic; }

  .btn-primary {
    width: 100%;
    height: 55px;
    background: var(--accent-color);
    color: white;
    border: none;
    border-radius: 30px;
    font-size: 1.1rem;
    font-weight: 700;
    cursor: pointer;
    box-shadow: 0 4px 15px rgba(56, 178, 172, 0.4);
    transition: transform 0.1s;
  }
  .btn-primary:active { transform: scale(0.98); }

  .info-popup {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 85%;
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(15px);
    -webkit-backdrop-filter: blur(15px);
    padding: 2rem;
    border-radius: 24px;
    box-shadow: 0 10px 40px rgba(0,0,0,0.2);
    z-index: 50;
    text-align: center;
  }
  .info-popup h4 { margin-bottom: 0.5rem; color: var(--accent-color); font-size: 1.2rem;}
  .info-popup p { font-size: 1rem; color: var(--text-secondary); margin-bottom: 1.5rem; line-height: 1.5; }
  
  .close-info {
    background: var(--text-secondary);
    color: white;
    border: none;
    padding: 10px 24px;
    border-radius: 30px;
    font-weight: 600;
    cursor: pointer;
  }
</style>