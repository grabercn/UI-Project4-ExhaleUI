<script>
  import { onMount } from 'svelte';
  import { scale, fade } from 'svelte/transition';

  // --- CONFIGURATION ---
  const modes = {
    sensory: {
      label: "5-4-3-2-1 Sense",
      desc: "Use your five senses to ground yourself in the physical world. Great for dissociation.",
      type: "checklist"
    },
    body: {
      label: "Body Scan",
      desc: "Progressive Muscle Relaxation (PMR). Tense and release muscles to stop physical shaking.",
      type: "checklist"
    },
    mental: {
      label: "Categories Game",
      desc: "Engage your prefrontal cortex (logic brain) to dampen the amygdala (panic brain).",
      type: "game"
    }
  };

  // --- DATA ---
  const sensorySteps = [
    { count: 5, label: "Things you see", desc: "A shadow, a color, a pattern.", checked: false },
    { count: 4, label: "Things you feel", desc: "Feet on floor, clothes on skin.", checked: false },
    { count: 3, label: "Things you hear", desc: "Traffic, birds, silence.", checked: false },
    { count: 2, label: "Things you smell", desc: "Soap, air, coffee.", checked: false },
    { count: 1, label: "Thing you taste", desc: "Toothpaste or imagine a lemon.", checked: false },
  ];

  const bodySteps = [
    { count: 1, label: "Clench Fists", desc: "Squeeze tight for 5s, then release.", checked: false },
    { count: 2, label: "Raise Shoulders", desc: "Push them to your ears, drop them.", checked: false },
    { count: 3, label: "Flex Toes", desc: "Curl them downward, then spread them.", checked: false },
    { count: 4, label: "Jaw Release", desc: "Open mouth wide, then relax jaw.", checked: false },
    { count: 5, label: "Deep Breath", desc: "Fill belly, sigh it out loudly.", checked: false },
  ];

  const categories = [
    "Movies seen recently", "Types of Cereal", "European Countries", 
    "Colors in this room", "Animals that swim", "Ice Cream flavors", 
    "Friends' names", "Pizza Toppings", "Things that are Blue"
  ];

  // --- STATE ---
  let selectedMode = 'sensory';
  let showInfo = false;
  let isFavorite = false;
  let activeInfoItem = null; // For the inline "i" buttons on list items
  
  // logic State
  let activeList = JSON.parse(JSON.stringify(sensorySteps));
  let currentCategory = categories[0];

  // --- LIFECYCLE ---
  onMount(() => {
    // load favorite
    const fav = localStorage.getItem('exhale_grounding_fav');
    if (fav && modes[fav]) {
      selectedMode = fav;
      isFavorite = true;
      changeMode(true); // Load data without resetting favorite
    }
  });

  // --- ACTIONS ---
  function changeMode(initialLoad = false) {
    if (!initialLoad) showInfo = false;
    
    // check if new selection is the favorite
    const saved = localStorage.getItem('exhale_grounding_fav');
    isFavorite = (saved === selectedMode);

    // reset data
    activeInfoItem = null;
    if (modes[selectedMode].type === 'checklist') {
      const source = selectedMode === 'sensory' ? sensorySteps : bodySteps;
      activeList = JSON.parse(JSON.stringify(source));
    } else {
      newCategory();
    }
  }

  function toggleFavorite() {
    if (isFavorite) {
      localStorage.removeItem('exhale_grounding_fav');
      isFavorite = false;
    } else {
      localStorage.setItem('exhale_grounding_fav', selectedMode);
      isFavorite = true;
    }
  }

  // checklist actions
  function toggleCheck(index) {
    activeList[index].checked = !activeList[index].checked;
  }

  function toggleItemInfo(e, index) {
    e.stopPropagation();
    activeInfoItem = activeInfoItem === index ? null : index;
  }

  function resetList() {
    activeList = activeList.map(s => ({...s, checked: false}));
    activeInfoItem = null;
  }

  // game actions
  function newCategory() {
    let newCat;
    do {
      newCat = categories[Math.floor(Math.random() * categories.length)];
    } while (newCat === currentCategory); // prevent all duplicates
    currentCategory = newCat;
  }
</script>

<div class="grounding-container">
  
  {#if showInfo}
    <div class="info-popup" transition:scale={{duration: 200}}>
      <h4>{modes[selectedMode].label}</h4>
      <p>{modes[selectedMode].desc}</p>
      <button class="close-info" on:click={() => showInfo = false}>Got it</button>
    </div>
  {/if}

  <div class="selector-row">
    <select bind:value={selectedMode} on:change={() => changeMode(false)}>
      {#each Object.entries(modes) as [key, mode]}
        <option value={key}>{mode.label}</option>
      {/each}
    </select>

    <button 
      class="icon-btn" 
      class:active={isFavorite}
      on:click={toggleFavorite}
      aria-label="Set as favorite">
      {isFavorite ? '★' : '☆'}
    </button>

    <button 
      class="icon-btn info-btn" 
      on:click={() => showInfo = !showInfo}
      aria-label="Mode Information">
      i
    </button>
  </div>

  <div class="content-wrapper">
    
    {#if modes[selectedMode].type === 'checklist'}
      <div class="list-wrapper" in:fade={{duration: 200}}>
        {#each activeList as step, i}
          <div 
            class="step-item" 
            class:done={step.checked}
            on:click={() => toggleCheck(i)}
          >
            <div class="count">{step.count}</div>
            <div class="label">{step.label}</div>
            
            <button class="item-info-btn" on:click={(e) => toggleItemInfo(e, i)}>i</button>
            
            {#if step.checked}<div class="check">✓</div>{/if}

            {#if activeInfoItem === i}
              <div class="inline-popup" transition:scale={{duration: 150, start: 0.9}}>
                {step.desc}
              </div>
            {/if}
          </div>
        {/each}
        
        <button class="action-link" on:click={resetList}>Reset List</button>
      </div>

    {:else}
      <div class="game-wrapper" in:fade={{duration: 200}}>
        <div class="card">
          <span class="sub">Name 3...</span>
          <h2>{currentCategory}</h2>
        </div>
        <button class="btn-primary" on:click={newCategory}>New Category</button>
      </div>
    {/if}

  </div>
</div>

<style>
  .grounding-container {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
    padding-top: 2rem;
    position: relative;
  }

  /* --- HEADER CONTROLS (shared with Breathing comp.) --- */
  .selector-row {
    display: flex;
    gap: 10px;
    height: 50px;
    flex-shrink: 0;
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
  }

  .icon-btn {
    width: 50px;
    height: 50px;
    border-radius: 12px;
    border: none;
    background: rgba(255,255,255,0.3);
    font-size: 1.2rem;
    color: var(--text-secondary);
    cursor: pointer;
    display: flex;
    justify-content: center;
    align-items: center;
    transition: all 0.2s;
  }

  .icon-btn:hover { background: rgba(255,255,255,0.5); }
  .icon-btn.active { color: #ecc94b; }
  .info-btn { font-family: serif; font-weight: bold; font-style: italic; }

  /* --- POPUPS --- */
  .info-popup {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 90%;
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(10px);
    padding: 1.5rem;
    border-radius: 20px;
    box-shadow: 0 10px 40px rgba(0,0,0,0.2);
    z-index: 20;
    text-align: center;
  }
  .info-popup h4 { margin-bottom: 0.5rem; color: var(--accent-color); }
  .info-popup p { font-size: 0.9rem; color: var(--text-secondary); margin-bottom: 1rem; line-height: 1.5; }
  
  .close-info {
    background: var(--text-secondary);
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 20px;
    font-size: 0.8rem;
    cursor: pointer;
  }

  /* --- CONTENT WRAPPER --- */
  .content-wrapper {
    flex: 1;
    overflow-y: auto;
    position: relative;
    padding-bottom: 1rem;
    /* Hide scrollbar */
    scrollbar-width: none;
  }

  /* --- LIST STYLES --- */
  .list-wrapper {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .step-item {
    position: relative;
    background: rgba(255, 255, 255, 0.4);
    padding: 15px;
    border-radius: 16px;
    display: flex;
    align-items: center;
    gap: 12px;
    cursor: pointer;
    transition: all 0.2s;
    user-select: none;
  }

  .step-item.done {
    opacity: 0.6;
    background: rgba(255, 255, 255, 0.2);
  }

  .count {
    background: var(--accent-color);
    color: white;
    width: 28px;
    height: 28px;
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    font-weight: 700;
    font-size: 0.9rem;
    flex-shrink: 0;
  }

  .label { flex: 1; font-weight: 600; font-size: 0.95rem; }

  .item-info-btn {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    border: 1px solid var(--text-secondary);
    background: transparent;
    color: var(--text-secondary);
    font-family: serif;
    font-style: italic;
    cursor: pointer;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 0.8rem;
    opacity: 0.5;
  }
  
  .check { color: var(--accent-color); font-weight: 800; }

  .inline-popup {
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    margin-top: 5px;
    background: #fff;
    padding: 12px;
    border-radius: 12px;
    box-shadow: 0 4px 15px rgba(0,0,0,0.1);
    z-index: 10;
    font-size: 0.9rem;
    color: var(--text-secondary);
    border: 1px solid rgba(0,0,0,0.05);
  }

  .action-link {
    background: transparent;
    border: none;
    color: var(--text-secondary);
    text-decoration: underline;
    cursor: pointer;
    align-self: center;
    margin-top: 10px;
  }

  /* --- GAME STYLES --- */
  .game-wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100%;
    gap: 2rem;
  }

  .card {
    background: rgba(255,255,255,0.6);
    padding: 2rem;
    border-radius: 24px;
    width: 100%;
    text-align: center;
    box-shadow: 0 4px 20px rgba(0,0,0,0.05);
  }

  .sub {
    text-transform: uppercase;
    letter-spacing: 1px;
    font-size: 0.8rem;
    color: var(--text-secondary);
    display: block;
    margin-bottom: 0.5rem;
  }

  h2 { font-size: 1.5rem; color: var(--text-primary); margin: 0; }

  .btn-primary {
    background: var(--accent-color);
    color: white;
    border: none;
    padding: 14px 28px;
    border-radius: 50px;
    font-size: 1.1rem;
    font-weight: 700;
    cursor: pointer;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    transition: transform 0.1s;
    width: 100%;
  }
  .btn-primary:active { transform: scale(0.98); }
</style>