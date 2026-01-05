<script>
  import forgePerks from '../forgePerks.json';
  import items from '../items.json';
  import bgImage from '/resources/webpageBgImg.jpg';

  // Transform the data into a table-friendly format
  let perkData = [];

  for (const [perkName, perkInfo] of Object.entries(forgePerks)) {
    perkData.push({
      name: perkName.replace('Perk_', ''),
      description: perkInfo.description,
      elite: perkInfo.elite,
      equipment: perkInfo.equipment.map(eq => eq.replace('Equipment.', '')),
      itemsRaw: perkInfo.items,
      items: Object.entries(perkInfo.items)
    });
  }

  // Sorting and filtering
  let sortColumn = 'name';
  let sortAscending = true;
  let filterEquipment = 'all';
  let filterElite = 'all';
  let searchTerm = '';

  $: equipmentTypes = ['all', ...new Set(Object.values(forgePerks).flatMap(p => p.equipment.map(eq => eq.replace('Equipment.', ''))))];

  $: filteredData = perkData
    .filter(perk => {
      const matchesSearch = searchTerm === '' ||
        perk.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
        perk.description.toLowerCase().includes(searchTerm.toLowerCase()) ||
        perk.items.some(([item]) => item.toLowerCase().includes(searchTerm.toLowerCase()));

      const matchesEquipment = filterEquipment === 'all' ||
        perk.equipment.includes(filterEquipment);

      const matchesElite = filterElite === 'all' ||
        (filterElite === 'elite' && perk.elite) ||
        (filterElite === 'normal' && !perk.elite);

      return matchesSearch && matchesEquipment && matchesElite;
    })
    .sort((a, b) => {
      let aVal = a[sortColumn];
      let bVal = b[sortColumn];

      if (sortColumn === 'elite') {
        aVal = a.elite ? 1 : 0;
        bVal = b.elite ? 1 : 0;
      }

      if (aVal < bVal) return sortAscending ? -1 : 1;
      if (aVal > bVal) return sortAscending ? 1 : -1;
      return 0;
    });

  function toggleSort(column) {
    if (sortColumn === column) {
      sortAscending = !sortAscending;
    } else {
      sortColumn = column;
      sortAscending = true;
    }
  }

  function getSortIcon(column) {
    if (sortColumn !== column) return '';
    return sortAscending ? ' ▲' : ' ▼';
  }

  function formatItemName(itemName) {
    // Try to get the friendly name from items.json
    if (items[itemName] && items[itemName].inGameName) {
      return items[itemName].inGameName;
    }
    // Fallback to formatted version if not found
    return itemName.replace(/_/g, ' ').replace(/^AI /, '').replace(/(\d+)$/, '');
  }

  // Map item names to image files
  function getItemImage(itemName) {
    // Get the friendly name and convert it to lowercase filename format
    const friendlyName = formatItemName(itemName);
    // Convert to lowercase, replace spaces with nothing, add .avif extension
    const imageFileName = friendlyName.toLowerCase().replace(/\s+/g, '').replace(/'/g, '') + '.avif';
    return imageFileName;
  }
</script>

<div class="app-wrapper" style="--bg-image: url({bgImage})">
<main>
  <h1>S2 Forge Recipes</h1>

  <div class="filters">
    <div class="filter-group">
      <label>
        Search:
        <input type="text" bind:value={searchTerm} placeholder="Search perks, items..." />
      </label>
    </div>

    <div class="filter-group">
      <label>
        Equipment Type:
        <select bind:value={filterEquipment}>
          {#each equipmentTypes as type}
            <option value={type}>{type === 'all' ? 'All Types' : type}</option>
          {/each}
        </select>
      </label>
    </div>

    <div class="filter-group">
      <label>
        Rarity:
        <select bind:value={filterElite}>
          <option value="all">All</option>
          <option value="normal">Normal</option>
          <option value="elite">Elite</option>
        </select>
      </label>
    </div>

    <div class="stats">
      Showing {filteredData.length} of {perkData.length} perks
    </div>
  </div>

  <div class="table-container">
    <table>
      <thead>
        <tr>
          <th class="perk-col" rowspan="2">Perk</th>
          <th class="items-col" colspan="4">Compatible Perk Items (one of the following)</th>
          <th class="compatibility-header" colspan="3">Compatibility</th>
        </tr>
        <tr class="subheader">
          <th colspan="4"></th>
          <th>Backpack</th>
          <th>Shield</th>
          <th>Helmet</th>
        </tr>
      </thead>
      <tbody>
        {#each filteredData as perk}
          <tr>
            <td class="perk-cell">
              <div class="perk-name">{perk.description}</div>
            </td>
            {#each perk.items.slice(0, 4) as [itemName, quantity], i}
              <td class="item-cell">
                <div class="item-content">
                  <span class="item-quantity">{quantity}</span>
                  <img src="resources/images/{getItemImage(itemName)}" alt={itemName} class="item-icon" />
                  <span class="item-name">{formatItemName(itemName)}</span>
                </div>
              </td>
            {/each}
            {#each Array(Math.max(0, 4 - perk.items.length)) as _}
              <td class="item-cell empty"></td>
            {/each}
            <td class="compat-cell">
              {#if perk.equipment.includes('Back Item')}
                <span class="checkmark">✓</span>
              {/if}
            </td>
            <td class="compat-cell">
              {#if perk.equipment.includes('Shield')}
                <span class="checkmark">✓</span>
              {/if}
            </td>
            <td class="compat-cell">
              {#if perk.equipment.includes('Helmet')}
                <span class="checkmark">✓</span>
              {/if}
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
</main>
</div>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
    color: #b0bec5;
  }

  .app-wrapper {
    background: #0a1929 var(--bg-image) no-repeat center center fixed;
    background-size: cover;
    min-height: 100vh;
    position: relative;
  }

  .app-wrapper::before {
    content: '';
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(10, 25, 41, 0.60);
    z-index: -1;
  }

  main {
    max-width: 95%;
    margin: 0 auto;
    padding: 2rem;
  }

  h1 {
    color: #64b5f6;
    text-align: center;
    font-size: 2.5rem;
    margin-bottom: 2rem;
    text-shadow: 0 0 10px rgba(100, 181, 246, 0.3);
  }

  .filters {
    background: #1a2332;
    padding: 1.5rem;
    border-radius: 4px;
    margin-bottom: 1.5rem;
    display: flex;
    gap: 1rem;
    flex-wrap: wrap;
    align-items: center;
    border: 1px solid #2d3a4f;
  }

  .filter-group {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
  }

  .filter-group label {
    font-weight: 600;
    font-size: 0.9rem;
    color: #90caf9;
  }

  .filter-group input,
  .filter-group select {
    padding: 0.5rem;
    border: 1px solid #2d3a4f;
    border-radius: 4px;
    font-size: 1rem;
    background: #0f1720;
    color: #b0bec5;
    transition: border-color 0.2s;
  }

  .filter-group input:focus,
  .filter-group select:focus {
    outline: none;
    border-color: #64b5f6;
  }

  .stats {
    margin-left: auto;
    font-weight: 600;
    color: #90caf9;
    padding: 0.5rem;
  }

  .table-container {
    background: #1a2332;
    border-radius: 4px;
    overflow-x: auto;
    border: 1px solid #2d3a4f;
  }

  table {
    width: 100%;
    border-collapse: collapse;
  }

  thead {
    background: #1a2838;
    color: #90caf9;
  }

  thead tr:first-child th {
    padding: 0.875rem 1rem;
    text-align: center;
    font-weight: 600;
    font-size: 0.9rem;
    border-right: 1px solid #2d3a4f;
    border-bottom: 1px solid #2d3a4f;
    background: #1a2838;
  }

  thead tr:first-child th.perk-col {
    text-align: left;
    vertical-align: middle;
  }

  thead tr.subheader th {
    padding: 0.75rem 0.5rem;
    text-align: center;
    font-weight: 600;
    font-size: 0.85rem;
    border-bottom: 2px solid #2d3a4f;
    border-right: 1px solid #2d3a4f;
    background: #1a2838;
  }

  thead tr.subheader th:last-child {
    border-right: none;
  }

  .perk-col {
    width: 20%;
    min-width: 200px;
  }

  .items-col {
    text-align: center;
  }

  .compatibility-header {
    text-align: center;
  }

  tbody tr {
    border-bottom: 1px solid #2d3a4f;
    transition: background-color 0.2s;
  }

  tbody tr:hover {
    background-color: #1f2937;
  }

  td {
    padding: 0.875rem;
    font-size: 0.9rem;
  }

  .perk-cell {
    background: #162030;
    border-right: 1px solid #2d3a4f;
    vertical-align: middle;
  }

  .perk-name {
    color: #90caf9;
    font-weight: 500;
    font-size: 0.95rem;
  }

  .item-cell {
    text-align: left;
    padding: 0.75rem 0.5rem;
    color: #b0bec5;
    border-right: 1px solid #2d3a4f;
    min-width: 180px;
  }

  .item-cell.empty {
    background: #0f1720;
  }

  .item-content {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .item-quantity {
    display: inline-block;
    min-width: 1.5rem;
    font-weight: 600;
    color: #64b5f6;
    flex-shrink: 0;
  }

  .item-icon {
    width: 32px;
    height: 32px;
    object-fit: contain;
    flex-shrink: 0;
  }

  .item-name {
    color: #b0bec5;
    flex: 1;
  }

  .compat-cell {
    text-align: center;
    background: #0f1720;
    border-right: 1px solid #2d3a4f;
    padding: 0.75rem;
  }

  .compat-cell:last-child {
    border-right: none;
  }

  .checkmark {
    color: #64b5f6;
    font-size: 1.2rem;
    font-weight: bold;
  }

  @media (max-width: 768px) {
    main {
      padding: 1rem;
    }

    h1 {
      font-size: 1.75rem;
    }

    .filters {
      flex-direction: column;
      align-items: stretch;
    }

    .stats {
      margin-left: 0;
      text-align: center;
    }

    table {
      font-size: 0.8rem;
    }

    th, td {
      padding: 0.5rem 0.25rem;
    }

    .item-cell {
      min-width: 120px;
    }
  }
</style>
