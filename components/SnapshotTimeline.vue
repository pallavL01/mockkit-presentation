<template>
  <div class="card" style="padding: 20px 28px; margin-top: 10px;">
    <div style="position: relative; padding: 44px 0;">
      <!-- Main timeline bar -->
      <div style="position: absolute; top: 50%; left: 16px; right: 16px; height: 8px; background: var(--slate-700); border-radius: 4px; transform: translateY(-50%);">
        <div style="position: absolute; right: -10px; top: -4px; width: 0; height: 0; border-left: 14px solid var(--slate-700); border-top: 8px solid transparent; border-bottom: 8px solid transparent;"></div>
      </div>
      <!-- Label -->
      <div style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: var(--slate-700); color: #e2e8f0; padding: 3px 12px; border-radius: 3px; font-size: 11px; font-weight: 700; z-index: 1;">
        Recordings →
      </div>
      <!-- Branch UP: bugfix-123 -->
      <div style="position: absolute; top: -6px; right: 60px; display: flex; flex-direction: column; align-items: center; gap: 4px;">
        <div style="display: flex; align-items: center; gap: 8px;">
          <div style="background: var(--card); border: 1px solid var(--card-border); border-radius: 4px; padding: 4px 10px; font-size: 10px; color: var(--slate-600);">
            snapshots activate "bugfix-123"
          </div>
          <div style="background: #6b7a96; color: white; padding: 5px 14px; border-radius: 14px; font-size: 11px; font-weight: 600;">
            save "bugfix-123"
          </div>
        </div>
        <div style="width: 2px; height: 14px; background: var(--slate-700);"></div>
      </div>
      <!-- Branch DOWN: baseline-v2 -->
      <div style="position: absolute; bottom: -6px; left: 50px; display: flex; flex-direction: column; align-items: center; gap: 4px;">
        <div style="width: 2px; height: 14px; background: var(--slate-700);"></div>
        <div style="display: flex; align-items: center; gap: 8px;">
          <div style="background: #6b7a96; color: white; padding: 5px 14px; border-radius: 14px; font-size: 11px; font-weight: 600;">
            save "baseline-v2"
          </div>
          <div style="background: var(--card); border: 1px solid var(--card-border); border-radius: 4px; padding: 4px 10px; font-size: 10px; color: var(--slate-600);">
            snapshots save "baseline-v2"
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
