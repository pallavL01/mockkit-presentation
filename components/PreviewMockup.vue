<template>
  <div style="background: white; border: 1px solid var(--card-border); border-radius: 10px; overflow: hidden;">
    <!-- Header -->
    <div style="display: flex; justify-content: space-between; align-items: center; padding: 8px 14px; border-bottom: 1px solid var(--card-border);">
      <strong style="font-size: 12px;">Preview UI</strong>
      <div style="display: flex; align-items: center; gap: 6px; font-size: 10px; color: var(--slate-500);">
        Seed
        <code style="background: var(--card); padding: 2px 8px; border-radius: 3px; font-size: 10px;">[12345]</code>
      </div>
    </div>
    <div style="display: grid; grid-template-columns: 120px 1fr; min-height: 160px;">
      <!-- Sidebar -->
      <div style="border-right: 1px solid var(--card-border); padding: 10px;">
        <div style="font-weight: 700; font-size: 9px; margin-bottom: 8px; color: var(--slate-500);">Route Explorer</div>
        <div style="display: flex; flex-direction: column; gap: 6px;">
          <div style="display: flex; align-items: center; gap: 4px;">
            <div style="background: var(--teal); color: white; padding: 1px 5px; border-radius: 3px; font-size: 8px; font-weight: 700;">GET</div>
            <span style="font-size: 10px;">/users</span>
          </div>
          <div style="display: flex; align-items: center; gap: 4px;">
            <div style="background: #6366f1; color: white; padding: 1px 5px; border-radius: 3px; font-size: 8px; font-weight: 700;">POST</div>
            <span style="font-size: 10px;">/chat</span>
            <span style="font-size: 10px; color: var(--slate-400);">~</span>
          </div>
        </div>
      </div>
      <!-- Main -->
      <div style="padding: 10px; font-size: 11px;">
        <div style="font-weight: 700; margin-bottom: 6px; font-size: 11px;">Live Streaming Viewer</div>
        <div style="display: flex; flex-direction: column; gap: 4px;">
          <div class="card-light" style="padding: 5px 8px; font-size: 9px; position: relative;">
            {"data": {"message": "Hello!"}}
            <div style="position: absolute; right: 6px; top: 50%; transform: translateY(-50%); width: 7px; height: 7px; border-radius: 50%; background: var(--teal);"></div>
          </div>
          <div class="card-light" style="padding: 5px 8px; font-size: 9px; position: relative;">
            {"data": {"message": "How are you?"}}
            <div style="position: absolute; right: 6px; top: 50%; transform: translateY(-50%); width: 7px; height: 7px; border-radius: 50%; background: var(--teal);"></div>
          </div>
          <div class="card-light" style="padding: 5px 8px; font-size: 9px; position: relative;">
            {"data": {"message": "Streaming..."}}
            <div style="position: absolute; right: 6px; top: 50%; transform: translateY(-50%); width: 7px; height: 7px; border-radius: 50%; background: var(--teal);"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
