<template>
  <div class="card" style="padding: 12px 18px; margin-top: 8px; overflow: visible; position: relative;">
    <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 24px; align-items: start;">
      <!-- LEFT: Steps 1-3 -->
      <div style="display: flex; flex-direction: column; gap: 6px; font-size: 12px;">
        <!-- Step 1 -->
        <div style="display: flex; align-items: start; gap: 8px;">
          <div style="min-width: 22px; height: 22px; border-radius: 50%; background: #0d9488; color: white; display: flex; align-items: center; justify-content: center; font-weight: 700; font-size: 11px; flex-shrink: 0;">1</div>
          <div>
            <div style="font-weight: 700; font-size: 12px; color: var(--slate-900);">Browse & Record</div>
            <div style="background: var(--card); border: 1px solid var(--card-border); border-radius: 4px; padding: 4px 8px; margin-top: 3px; font-size: 10px; color: var(--slate-600);">
              Browse <strong style="color: var(--slate-900);">www.something.com</strong> — extension captures API calls
            </div>
          </div>
        </div>
        <div style="text-align: center; color: var(--slate-400); font-size: 14px; line-height: 1;">↓</div>
        <!-- Step 2 -->
        <div style="display: flex; align-items: start; gap: 8px;">
          <div style="min-width: 22px; height: 22px; border-radius: 50%; background: #0d9488; color: white; display: flex; align-items: center; justify-content: center; font-weight: 700; font-size: 11px; flex-shrink: 0;">2</div>
          <div>
            <div style="font-weight: 700; font-size: 12px; color: var(--slate-900);">Push to MockKit</div>
            <div style="background: var(--card); border: 1px solid var(--card-border); border-radius: 4px; padding: 4px 8px; margin-top: 3px; font-size: 10px; color: var(--slate-600);">
              Send to <strong style="color: var(--slate-900);">POST /__mock__/recordings</strong>
            </div>
          </div>
        </div>
        <div style="text-align: center; color: var(--slate-400); font-size: 14px; line-height: 1;">↓</div>
        <!-- Step 3 -->
        <div style="display: flex; align-items: start; gap: 8px;">
          <div style="min-width: 22px; height: 22px; border-radius: 50%; background: #0d9488; color: white; display: flex; align-items: center; justify-content: center; font-weight: 700; font-size: 11px; flex-shrink: 0;">3</div>
          <div>
            <div style="font-weight: 700; font-size: 12px; color: var(--slate-900);">Enable Live Mocking</div>
            <div style="background: var(--card); border: 1px solid var(--card-border); border-radius: 4px; padding: 4px 8px; margin-top: 3px; font-size: 10px; color: var(--slate-600);">
              api.something.com → localhost:3000
            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT: Step 4 flow -->
      <div style="display: flex; flex-direction: column; align-items: center; gap: 4px;">
        <div style="display: flex; align-items: center; gap: 6px; margin-bottom: 2px;">
          <div style="min-width: 22px; height: 22px; border-radius: 50%; background: #0d9488; color: white; display: flex; align-items: center; justify-content: center; font-weight: 700; font-size: 11px;">4</div>
          <span style="font-weight: 700; font-size: 12px; color: var(--slate-900);">Requests Intercepted</span>
        </div>
        <div style="background: #e2e6ee; border: 2px solid var(--slate-700); border-radius: 6px; padding: 8px 18px; text-align: center; width: 170px;">
          <div style="font-weight: 700; font-size: 12px; color: var(--slate-900);">Browser</div>
          <div style="font-size: 9px; color: var(--slate-500);">fetch('api.something.com')</div>
        </div>
        <div style="color: var(--slate-400); font-size: 14px; line-height: 1;">↓</div>
        <div style="background: var(--red-bg); border: 1px solid #fecaca; border-radius: 6px; padding: 6px 14px; text-align: center; width: 170px;">
          <div style="font-weight: 700; font-size: 10px; color: var(--red);">declarativeNetRequest</div>
          <div style="font-size: 9px; color: var(--red);">REDIRECT</div>
        </div>
        <div style="color: var(--slate-400); font-size: 14px; line-height: 1;">↓</div>
        <div style="background: #0d9488; border-radius: 6px; padding: 8px 18px; text-align: center; width: 170px;">
          <div style="font-weight: 700; font-size: 12px; color: white;">MockKit Server</div>
          <div style="font-size: 9px; color: #ccfbf1;">localhost:3000</div>
        </div>
        <div style="color: var(--slate-400); font-size: 14px; line-height: 1;">↓</div>
        <div style="background: #dcfce7; border: 1px solid #bbf7d0; border-radius: 5px; padding: 5px 14px; text-align: center; width: 170px;">
          <div style="font-weight: 700; font-size: 10px; color: var(--green);">Recorded Response</div>
        </div>
      </div>
    </div>

    <!-- SVG arrow overlay: from step 3 (bottom-left) to step 4 header (top-right) -->
    <svg viewBox="0 0 700 280" style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none;">
      <defs>
        <marker id="arr" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
          <polygon points="0 0, 10 3.5, 0 7" fill="#0d9488" />
        </marker>
      </defs>
      <!-- Curve from step 3 right edge to left of step 4 badge -->
      <path d="M 280 185 C 330 185, 420 28, 470 28" stroke="#0d9488" stroke-width="2" stroke-dasharray="6 4" fill="none" marker-end="url(#arr)" />
    </svg>
  </div>
</template>
