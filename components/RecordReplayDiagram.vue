<template>
  <div class="card" style="padding: 14px 20px; margin-top: 8px;">
    <div style="display: flex; align-items: center; justify-content: center; gap: 12px; flex-wrap: wrap; font-size: 13px;">
      <!-- Step 1: Client -->
      <div style="position: relative;">
        <div style="position: absolute; top: -10px; left: -6px; width: 20px; height: 20px; border-radius: 50%; background: #0d9488; color: white; display: flex; align-items: center; justify-content: center; font-size: 10px; font-weight: 700;">1</div>
        <div style="background: var(--card); border: 1px solid var(--card-border); border-radius: 8px; padding: 10px 20px; text-align: center;">
          <div style="font-weight: 700; font-size: 13px; color: var(--slate-900);">Client</div>
        </div>
      </div>
      <div style="font-size: 18px; color: var(--slate-400);">➜</div>
      <!-- Step 2: MockKit Proxy -->
      <div style="position: relative;">
        <div style="position: absolute; top: -10px; left: -6px; width: 20px; height: 20px; border-radius: 50%; background: #0d9488; color: white; display: flex; align-items: center; justify-content: center; font-size: 10px; font-weight: 700;">2</div>
        <div style="background: #e2e6ee; border: 2px solid var(--slate-700); border-radius: 10px; padding: 10px 20px; text-align: center;">
          <div style="font-weight: 700; font-size: 13px; color: var(--slate-900);">MockKit Proxy</div>
          <div style="font-size: 10px; color: var(--slate-500);">(Port 8080)</div>
        </div>
      </div>
      <div style="display: flex; flex-direction: column; gap: 2px; font-size: 16px; color: var(--slate-400);">
        <span>➜</span>
        <span>⬅</span>
      </div>
      <!-- Step 3: Real API -->
      <div style="position: relative;">
        <div style="position: absolute; top: -10px; left: -6px; width: 20px; height: 20px; border-radius: 50%; background: #0d9488; color: white; display: flex; align-items: center; justify-content: center; font-size: 10px; font-weight: 700;">3</div>
        <div style="background: var(--card); border: 1px solid var(--card-border); border-radius: 8px; padding: 10px 20px; text-align: center;">
          <div style="font-weight: 700; font-size: 13px; color: var(--slate-900);">Real API</div>
        </div>
      </div>
    </div>
    <!-- Step 4 -->
    <div style="display: flex; align-items: center; justify-content: center; margin-top: 10px; gap: 8px;">
      <div style="font-size: 16px; color: var(--slate-400);">↪</div>
      <div style="position: relative;">
        <div style="position: absolute; top: -10px; left: -6px; width: 20px; height: 20px; border-radius: 50%; background: #0d9488; color: white; display: flex; align-items: center; justify-content: center; font-size: 10px; font-weight: 700;">4</div>
        <div style="background: var(--card); border: 1px solid var(--card-border); border-radius: 8px; padding: 8px 20px; display: flex; align-items: center; gap: 8px;">
          <span style="font-size: 16px;">🗄</span>
          <div>
            <div style="font-weight: 700; font-size: 12px; color: var(--slate-900);">Response Captured</div>
            <div style="font-size: 9px; color: var(--slate-500);">Stored for deterministic replay</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
