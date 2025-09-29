<script>
  import { onMount, onDestroy } from 'svelte';
      
  let diceValue = 1;
  let isRolling = false;
  let motionEnabled = false; // steuert Sichtbarkeit des Enable-Buttons

  function rollDice() {
    if (isRolling) return;
    isRolling = true;

    let count = 0;
    const roll = setInterval(() => {
      diceValue = Math.floor(Math.random() * 6) + 1;
      count++;
      if (count > 8) {
        clearInterval(roll);
        isRolling = false;
      }
    }, 80);
  }

  function handleMotion(event) {
    const acc = event.accelerationIncludingGravity;
    if (!acc) return;
    const shake = Math.abs(acc.x) + Math.abs(acc.y) + Math.abs(acc.z);
    if (shake > 35 && !isRolling) {
      rollDice();
    }
  }

  async function startShake() {
    if (typeof DeviceMotionEvent !== 'undefined' &&
        typeof DeviceMotionEvent.requestPermission === 'function') {
      try {
        const response = await DeviceMotionEvent.requestPermission();
        if (response === 'granted') {
          window.addEventListener('devicemotion', handleMotion);
          motionEnabled = true;
        }
      } catch {
        alert('Need motion access to detect shaking');
      }
    } else if (typeof DeviceMotionEvent !== 'undefined') {
      window.addEventListener('devicemotion', handleMotion);
      motionEnabled = true;
    }
  }

  onMount(() => {
    // Wenn es die Permission-API nicht gibt, direkt aktivieren
    if (typeof DeviceMotionEvent !== 'undefined' &&
        typeof DeviceMotionEvent.requestPermission !== 'function') {
      window.addEventListener('devicemotion', handleMotion);
      motionEnabled = true;
    }
  });

  onDestroy(() => {
    if (typeof window !== 'undefined') {
      window.removeEventListener('devicemotion', handleMotion);
    }
  });
</script>

<div class="min-h-screen bg-gray-100 flex flex-col items-center justify-center p-5 text-gray-800 text-center">
  <h1 class="text-2xl font-semibold mb-8">Shake to Roll</h1>
  
  <!-- KORREKTE KLASSENBINDUNG -->
  <div class="dice-box" class:animate-shake={isRolling}>
    {diceValue}
  </div>

  <div class="flex flex-col gap-3 mt-8">
    <button on:click={rollDice} class="btn-primary" disabled={isRolling}>
      {isRolling ? 'Rolling...' : 'Tap to Roll'}
    </button>

    {#if !motionEnabled}
      <button on:click={startShake} class="btn-secondary">
        Enable Shake
      </button>
    {/if}
  </div>

  <p class="mt-6 opacity-70">Shake your phone to roll the dice</p>
</div>

<style>
  .dice-box {
    width: 90px;
    height: 90px;
    border-radius: 0.75rem;
    background: white;
    border: 2px solid #e5e7eb;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 2rem;
    font-weight: bold;
    color: #374151;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    transition: transform 0.2s;
  }

  .btn-primary {
    background: #2563eb;
    color: white;
    padding: 0.6rem 1.5rem;
    border-radius: 0.5rem;
    font-size: 1rem;
    font-weight: 500;
    cursor: pointer;
    transition: background 0.2s;
  }
  .btn-primary:hover { background: #1d4ed8; }

  .btn-secondary {
    background: #facc15;
    color: #111;
    padding: 0.6rem 1.5rem;
    border-radius: 0.5rem;
    font-size: 1rem;
    font-weight: 500;
    cursor: pointer;
    transition: background 0.2s;
  }
  .btn-secondary:hover { background: #eab308; }

  @keyframes shake {
    0% { transform: rotate(0deg); }
    25% { transform: rotate(-8deg); }
    50% { transform: rotate(8deg); }
    75% { transform: rotate(-6deg); }
    100% { transform: rotate(0deg); }
  }
  .animate-shake {
    animation: shake 0.5s ease-in-out;
  }
</style>
