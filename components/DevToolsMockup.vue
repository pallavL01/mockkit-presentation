<template>
  <div style="background: white; border: 1px solid var(--card-border); border-radius: 10px; overflow: hidden; font-size: 12px;">
    <!-- Tabs -->
    <div style="display: flex; gap: 0; border-bottom: 1px solid var(--card-border); padding: 0 10px;">
      <div style="padding: 6px 10px; color: var(--slate-500); font-size: 10px;">Network</div>
      <div style="padding: 6px 10px; color: var(--slate-900); font-weight: 700; font-size: 10px; border-bottom: 2px solid var(--teal);">MockKit</div>
      <div style="padding: 6px 10px; color: var(--slate-500); font-size: 10px;">Console</div>
    </div>
    <div style="padding: 10px;">
      <!-- Diff -->
      <div style="font-weight: 700; font-size: 11px; margin-bottom: 6px;">Smart Diffing</div>
      <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 6px; font-size: 10px;">
        <div>
          <div style="font-weight: 600; margin-bottom: 3px; font-size: 9px;">Live Response</div>
          <div style="background: var(--card); padding: 6px; border-radius: 4px; font-size: 9px; line-height: 1.4; word-break: break-all;">{ "id": 123, "data": { "key": <span style="background: #fecaca; color: #b91c1c; padding: 0 2px; border-radius: 2px;">"valueA"</span> } }</div>
        </div>
        <div>
          <div style="font-weight: 600; margin-bottom: 3px; font-size: 9px;">Mock Response</div>
          <div style="background: var(--card); padding: 6px; border-radius: 4px; font-size: 9px; line-height: 1.4; word-break: break-all;">{ "id": 123, "data": { "key": <span style="background: #bbf7d0; color: #15803d; padding: 0 2px; border-radius: 2px;">"valueB"</span> } }</div>
        </div>
      </div>
      <!-- GraphQL -->
      <div style="font-weight: 700; font-size: 11px; margin: 8px 0 4px;">GraphQL Intelligence</div>
      <div style="display: flex; flex-direction: column; gap: 3px;">
        <div class="card-light" style="padding: 4px 8px; font-size: 9px;">
          <strong>Query:</strong> GetUser <span style="color: var(--slate-400);">userId: '123'</span>
        </div>
        <div class="card-light" style="padding: 4px 8px; font-size: 9px;">
          <strong>Mutation:</strong> UpdateCart <span style="color: var(--slate-400);">cartId: 'abc'</span>
        </div>
      </div>
    </div>
  </div>
</template>
