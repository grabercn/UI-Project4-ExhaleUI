<script>
  import Breathing from './lib/Breathing.svelte';
  import Grounding from './lib/Grounding.svelte';
  
  let activeTab = 'breathing';
</script>

<div class="app-shell">
  <main class="glass-card">
    
    <header>
      <div class="logo">Exhale <span>UI</span></div>
      
      <div class="tabs">
        <button 
          class:active={activeTab === 'breathing'} 
          on:click={() => activeTab = 'breathing'}>
          Breathe
        </button>
        <button 
          class:active={activeTab === 'grounding'} 
          on:click={() => activeTab = 'grounding'}>
          Ground
        </button>
      </div>
    </header>

    <div class="content-viewport">
      {#if activeTab === 'breathing'}
        <Breathing />
      {:else}
        <Grounding />
      {/if}
    </div>

  </main>
</div>

<style>
  /* --- DESKTOP ver --- */
  .app-shell {
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    background: transparent; /* bg is handled by body in app.css */
  }

  .glass-card {
    background: var(--glass-bg);
    backdrop-filter: blur(12px);
    -webkit-backdrop-filter: blur(12px);
    border: 1px solid var(--glass-border);
    box-shadow: var(--shadow);
    
    width: 380px;
    height: 700px;
    border-radius: 30px;
    
    display: flex;
    flex-direction: column;
    overflow: hidden;
    position: relative;
  }

  /* --- HEADER and TABS --- */
  header {
    padding: 1.5rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid rgba(255,255,255,0.3);
    flex-shrink: 0;
  }

  .logo { font-size: 1.5rem; font-weight: 700; color: var(--text-primary); }
  .logo span { color: var(--accent-color); }

  .tabs {
    display: flex;
    background: rgba(255,255,255,0.3);
    border-radius: 20px;
    padding: 4px;
  }

  .tabs button {
    background: transparent;
    border: none;
    padding: 8px 16px;
    border-radius: 16px;
    font-size: 0.9rem;
    font-weight: 600;
    color: var(--text-secondary);
    transition: 0.2s;
  }

  .tabs button.active {
    background: #fff;
    color: var(--accent-color);
    box-shadow: 0 2px 5px rgba(0,0,0,0.05);
  }

  .content-viewport {
    flex: 1;
    overflow-y: auto;
    padding: 1.5rem;
    position: relative;
    /* hide the scrollbar but allow scrolling */
    scrollbar-width: none; 
    -ms-overflow-style: none;
  }
  .content-viewport::-webkit-scrollbar { display: none; }

  /* --- MOBILE FULLSCREEN --- */
  @media (max-width: 600px) {
    .app-shell {
      padding: 0;
      width: 100%;
      height: 100%;
      display: block; /* remove the flex centering */
    }

    .glass-card {
      width: 100%;
      height: 100dvh; /* dynamic viewport height style */
      max-width: none;
      border-radius: 0;
      border: none;
      box-shadow: none;
    }

    header {
      padding-top: max(1.5rem, env(safe-area-inset-top)); /* move below the status bar on mobile */
    }
  }
</style>