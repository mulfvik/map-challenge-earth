<script>
  import { formatNumber, formatCurrency } from '../data/countryData.js';
  
  export let countryData = [];
  export let selectedCountry;
  export let totalPopulation;
  export let totalGDP;
  export let averageLifeExpectancy;
  
  $: sortedByPopulation = [...countryData].sort((a, b) => b.population - a.population);
  $: sortedByGDP = [...countryData].sort((a, b) => b.gdp - a.gdp);
  $: sortedByLifeExpectancy = [...countryData].sort((a, b) => b.lifeExpectancy - a.lifeExpectancy);
  $: sortedByArea = [...countryData].sort((a, b) => b.area - a.area);
  $: sortedByDensity = [...countryData].sort((a, b) => b.density - a.density);
  $: sortedByLiteracy = [...countryData].sort((a, b) => b.literacy - a.literacy);
  $: sortedByUnemployment = [...countryData].sort((a, b) => a.unemployment - b.unemployment); // Lower is better
  $: sortedByEPI = [...countryData].sort((a, b) => b.epiScore - a.epiScore);
  $: sortedByRenewable = [...countryData].sort((a, b) => b.renewableEnergy - a.renewableEnergy);
  $: sortedByForest = [...countryData].sort((a, b) => b.forestCoverage - a.forestCoverage);
  $: sortedByCO2 = [...countryData].sort((a, b) => a.co2Emissions - b.co2Emissions); // Lower is better
  
  let activeTab = 'overview';
  
  function selectCountry(country) {
    selectedCountry.set(country);
  }

  function handleKeydown(event, country) {
    if (event.key === 'Enter' || event.key === ' ') {
      event.preventDefault();
      selectCountry(country);
    }
  }
</script>

<div class="stats-container">
  <div class="tabs">
    <button 
      class="tab" 
      class:active={activeTab === 'overview'}
      on:click={() => activeTab = 'overview'}
    >
      Overview
    </button>
    <button 
      class="tab" 
      class:active={activeTab === 'population'}
      on:click={() => activeTab = 'population'}
    >
      Population
    </button>
    <button 
      class="tab" 
      class:active={activeTab === 'economy'}
      on:click={() => activeTab = 'economy'}
    >
      Economy
    </button>
    <button 
      class="tab" 
      class:active={activeTab === 'geography'}
      on:click={() => activeTab = 'geography'}
    >
      Geography
    </button>
    <button 
      class="tab" 
      class:active={activeTab === 'social'}
      on:click={() => activeTab = 'social'}
    >
      Social
    </button>
    <button 
      class="tab" 
      class:active={activeTab === 'environment'}
      on:click={() => activeTab = 'environment'}
    >
      Environment
    </button>
  </div>
  
  {#if activeTab === 'overview'}
    <div class="tab-content">
      <div class="stat-card">
        <h3>Global Population</h3>
        <div class="value">{formatNumber(totalPopulation)}</div>
        <div class="label">Total People</div>
      </div>
      
      <div class="stat-card">
        <h3>Global GDP</h3>
        <div class="value">{formatCurrency(totalGDP)}</div>
        <div class="label">Combined Economy</div>
      </div>
      
      <div class="stat-card">
        <h3>Average Life Expectancy</h3>
        <div class="value">{averageLifeExpectancy.toFixed(1)}</div>
        <div class="label">Years</div>
      </div>
      
      {#if $selectedCountry}
        <div class="selected-country">
          <h3>Selected Country</h3>
          <div class="country-details">
            <h4>{$selectedCountry.name}</h4>
            <div class="detail-grid">
              <div class="detail-item">
                <span class="detail-label">Capital:</span>
                <span class="detail-value">{$selectedCountry.capital}</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">Continent:</span>
                <span class="detail-value">{$selectedCountry.continent}</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">Population:</span>
                <span class="detail-value">{formatNumber($selectedCountry.population)}</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">GDP:</span>
                <span class="detail-value">{formatCurrency($selectedCountry.gdp)}</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">Life Expectancy:</span>
                <span class="detail-value">{$selectedCountry.lifeExpectancy} years</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">Area:</span>
                <span class="detail-value">{formatNumber($selectedCountry.area)} km²</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">Density:</span>
                <span class="detail-value">{$selectedCountry.density} people/km²</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">Literacy Rate:</span>
                <span class="detail-value">{$selectedCountry.literacy}%</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">Unemployment:</span>
                <span class="detail-value">{$selectedCountry.unemployment}%</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">CO₂ Emissions:</span>
                <span class="detail-value">{$selectedCountry.co2Emissions} tonnes/capita</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">Renewable Energy:</span>
                <span class="detail-value">{$selectedCountry.renewableEnergy}%</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">Forest Coverage:</span>
                <span class="detail-value">{$selectedCountry.forestCoverage}%</span>
              </div>
              <div class="detail-item">
                <span class="detail-label">Environmental Score:</span>
                <span class="detail-value">{$selectedCountry.epiScore}/100</span>
              </div>
            </div>
          </div>
        </div>
      {:else}
        <div class="instruction">
          <p>Click on a country in the globe to see detailed information</p>
        </div>
      {/if}
    </div>
  {/if}
  
  {#if activeTab === 'population'}
    <div class="tab-content">
      <h3>Countries by Population</h3>
      <div class="country-list">
        {#each sortedByPopulation as country, index}
          <div 
            class="country-item" 
            class:selected={$selectedCountry?.iso === country.iso}
            on:click={() => selectCountry(country)}
            on:keydown={(event) => handleKeydown(event, country)}
            role="button"
            tabindex="0"
            aria-label="Select {country.name}"
          >
            <div class="country-info">
              <span class="rank">#{index + 1}</span>
              <span class="country-name">{country.name}</span>
            </div>
            <span class="country-value">{formatNumber(country.population)}</span>
          </div>
        {/each}
      </div>
    </div>
  {/if}
  
  {#if activeTab === 'economy'}
    <div class="tab-content">
      <h3>Countries by GDP</h3>
      <div class="country-list">
        {#each sortedByGDP as country, index}
          <div 
            class="country-item"
            class:selected={$selectedCountry?.iso === country.iso}
            on:click={() => selectCountry(country)}
            on:keydown={(event) => handleKeydown(event, country)}
            role="button"
            tabindex="0"
            aria-label="Select {country.name}"
          >
            <div class="country-info">
              <span class="rank">#{index + 1}</span>
              <span class="country-name">{country.name}</span>
            </div>
            <span class="country-value">{formatCurrency(country.gdp)}</span>
          </div>
        {/each}
      </div>
    </div>
  {/if}
  
  {#if activeTab === 'geography'}
    <div class="tab-content">
      <h3>Countries by Area</h3>
      <div class="country-list">
        {#each sortedByArea as country, index}
          <div 
            class="country-item"
            class:selected={$selectedCountry?.iso === country.iso}
            on:click={() => selectCountry(country)}
            on:keydown={(event) => handleKeydown(event, country)}
            role="button"
            tabindex="0"
            aria-label="Select {country.name}"
          >
            <div class="country-info">
              <span class="rank">#{index + 1}</span>
              <span class="country-name">{country.name}</span>
            </div>
            <span class="country-value">{formatNumber(country.area)} km²</span>
          </div>
        {/each}
      </div>
      
      <h3 style="margin-top: 2rem;">Countries by Population Density</h3>
      <div class="country-list">
        {#each sortedByDensity as country, index}
          <div 
            class="country-item"
            class:selected={$selectedCountry?.iso === country.iso}
            on:click={() => selectCountry(country)}
            on:keydown={(event) => handleKeydown(event, country)}
            role="button"
            tabindex="0"
            aria-label="Select {country.name}"
          >
            <div class="country-info">
              <span class="rank">#{index + 1}</span>
              <span class="country-name">{country.name}</span>
            </div>
            <span class="country-value">{country.density} people/km²</span>
          </div>
        {/each}
      </div>
    </div>
  {/if}
  
  {#if activeTab === 'social'}
    <div class="tab-content">
      <h3>Countries by Life Expectancy</h3>
      <div class="country-list">
        {#each sortedByLifeExpectancy as country, index}
          <div 
            class="country-item"
            class:selected={$selectedCountry?.iso === country.iso}
            on:click={() => selectCountry(country)}
            on:keydown={(event) => handleKeydown(event, country)}
            role="button"
            tabindex="0"
            aria-label="Select {country.name}"
          >
            <div class="country-info">
              <span class="rank">#{index + 1}</span>
              <span class="country-name">{country.name}</span>
            </div>
            <span class="country-value">{country.lifeExpectancy} years</span>
          </div>
        {/each}
      </div>
      
      <h3 style="margin-top: 2rem;">Countries by Literacy Rate</h3>
      <div class="country-list">
        {#each sortedByLiteracy as country, index}
          <div 
            class="country-item"
            class:selected={$selectedCountry?.iso === country.iso}
            on:click={() => selectCountry(country)}
            on:keydown={(event) => handleKeydown(event, country)}
            role="button"
            tabindex="0"
            aria-label="Select {country.name}"
          >
            <div class="country-info">
              <span class="rank">#{index + 1}</span>
              <span class="country-name">{country.name}</span>
            </div>
            <span class="country-value">{country.literacy}%</span>
          </div>
        {/each}
      </div>
      
      <h3 style="margin-top: 2rem;">Countries by Employment (Lowest Unemployment)</h3>
      <div class="country-list">
        {#each sortedByUnemployment as country, index}
          <div 
            class="country-item"
            class:selected={$selectedCountry?.iso === country.iso}
            on:click={() => selectCountry(country)}
            on:keydown={(event) => handleKeydown(event, country)}
            role="button"
            tabindex="0"
            aria-label="Select {country.name}"
          >
            <div class="country-info">
              <span class="rank">#{index + 1}</span>
              <span class="country-name">{country.name}</span>
            </div>
            <span class="country-value">{country.unemployment}%</span>
          </div>
        {/each}
      </div>
    </div>
  {/if}
  
  {#if activeTab === 'environment'}
    <div class="tab-content">
      <h3>Countries by Environmental Performance</h3>
      <div class="country-list">
        {#each sortedByEPI as country, index}
          <div 
            class="country-item"
            class:selected={$selectedCountry?.iso === country.iso}
            on:click={() => selectCountry(country)}
            on:keydown={(event) => handleKeydown(event, country)}
            role="button"
            tabindex="0"
            aria-label="Select {country.name}"
          >
            <div class="country-info">
              <span class="rank">#{index + 1}</span>
              <span class="country-name">{country.name}</span>
            </div>
            <span class="country-value">{country.epiScore}/100</span>
          </div>
        {/each}
      </div>
      
      <h3 style="margin-top: 2rem;">Countries by Renewable Energy Usage</h3>
      <div class="country-list">
        {#each sortedByRenewable as country, index}
          <div 
            class="country-item"
            class:selected={$selectedCountry?.iso === country.iso}
            on:click={() => selectCountry(country)}
            on:keydown={(event) => handleKeydown(event, country)}
            role="button"
            tabindex="0"
            aria-label="Select {country.name}"
          >
            <div class="country-info">
              <span class="rank">#{index + 1}</span>
              <span class="country-name">{country.name}</span>
            </div>
            <span class="country-value">{country.renewableEnergy}%</span>
          </div>
        {/each}
      </div>
      
      <h3 style="margin-top: 2rem;">Countries by Forest Coverage</h3>
      <div class="country-list">
        {#each sortedByForest as country, index}
          <div 
            class="country-item"
            class:selected={$selectedCountry?.iso === country.iso}
            on:click={() => selectCountry(country)}
            on:keydown={(event) => handleKeydown(event, country)}
            role="button"
            tabindex="0"
            aria-label="Select {country.name}"
          >
            <div class="country-info">
              <span class="rank">#{index + 1}</span>
              <span class="country-name">{country.name}</span>
            </div>
            <span class="country-value">{country.forestCoverage}%</span>
          </div>
        {/each}
      </div>
      
      <h3 style="margin-top: 2rem;">Countries by Lowest CO₂ Emissions</h3>
      <div class="country-list">
        {#each sortedByCO2 as country, index}
          <div 
            class="country-item"
            class:selected={$selectedCountry?.iso === country.iso}
            on:click={() => selectCountry(country)}
            on:keydown={(event) => handleKeydown(event, country)}
            role="button"
            tabindex="0"
            aria-label="Select {country.name}"
          >
            <div class="country-info">
              <span class="rank">#{index + 1}</span>
              <span class="country-name">{country.name}</span>
            </div>
            <span class="country-value">{country.co2Emissions} tonnes/capita</span>
          </div>
        {/each}
      </div>
    </div>
  {/if}
</div>

<style>
  .stats-container {
    height: 100%;
    display: flex;
    flex-direction: column;
  }
  
  .tabs {
    display: flex;
    border-bottom: 2px solid var(--border-color);
    margin-bottom: 24px;
    background: var(--bg-card);
    border-radius: 12px;
    padding: 4px;
    overflow: hidden;
  }
  
  .tab {
    flex: 1;
    padding: 14px 20px;
    background: none;
    border: none;
    font-weight: 600;
    font-size: 0.9rem;
    color: var(--text-secondary);
    cursor: pointer;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    border-radius: 8px;
    position: relative;
    overflow: hidden;
  }
  
  .tab::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: linear-gradient(135deg, var(--accent-primary), var(--accent-secondary));
    opacity: 0;
    transition: opacity 0.3s ease;
  }
  
  .tab:hover {
    color: var(--text-primary);
    background: var(--hover-bg);
    transform: translateY(-1px);
  }
  
  .tab.active {
    color: var(--bg-primary);
    background: linear-gradient(135deg, var(--text-muted), var(--accent-primary));
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    font-weight: 600;
  }
  
  .tab.active::before {
    opacity: 1;
  }
  
  .tab-content {
    flex: 1;
    overflow-y: auto;
  }
  
  .country-list {
    max-height: 500px;
    overflow-y: auto;
    scrollbar-width: thin;
    scrollbar-color: var(--accent-primary) transparent;
  }
  
  .country-list::-webkit-scrollbar {
    width: 6px;
  }
  
  .country-list::-webkit-scrollbar-track {
    background: transparent;
  }
  
  .country-list::-webkit-scrollbar-thumb {
    background: var(--accent-primary);
    border-radius: 3px;
  }
  
  .country-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 16px 20px;
    margin-bottom: 12px;
    background: var(--bg-card);
    border: 1px solid var(--border-color);
    border-radius: 12px;
    cursor: pointer;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    position: relative;
    overflow: hidden;
  }
  
  .country-item::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: linear-gradient(135deg, var(--text-muted), var(--accent-primary));
    opacity: 0;
    transition: opacity 0.3s ease;
  }
  
  .country-item:hover {
    background: var(--hover-bg);
    transform: translateX(8px) scale(1.02);
    box-shadow: var(--shadow-lg);
    border-color: var(--accent-primary);
  }
  
  .country-item:hover::before {
    opacity: 0.1;
  }
  
  .country-item.selected {
    background: linear-gradient(135deg, var(--text-muted), var(--accent-primary));
    color: white;
    border-color: var(--accent-primary);
    box-shadow: var(--shadow-xl);
  }
  
  .country-item.selected::before {
    opacity: 1;
  }
  
  .country-info {
    display: flex;
    align-items: center;
    gap: 12px;
    position: relative;
    z-index: 1;
  }
  
  .rank {
    font-size: 0.8rem;
    font-weight: 700;
    color: var(--text-muted);
    min-width: 28px;
    text-align: center;
    background: var(--bg-glass);
    padding: 4px 8px;
    border-radius: 6px;
    border: 1px solid var(--border-color);
  }
  
  .country-item.selected .rank {
    color: rgba(255, 255, 255, 0.9);
    background: rgba(255, 255, 255, 0.1);
    border-color: rgba(255, 255, 255, 0.2);
  }
  
  .country-name {
    font-weight: 600;
    color: var(--text-primary);
    position: relative;
    z-index: 1;
  }
  
  .country-item.selected .country-name {
    color: white;
  }
  
  .country-value {
    font-weight: 700;
    color: var(--text-secondary);
    position: relative;
    z-index: 1;
  }
  
  .country-item.selected .country-value {
    color: white;
  }
  
  .selected-country {
    margin-top: 24px;
    padding: 28px;
    background: linear-gradient(135deg, var(--bg-tertiary), var(--text-muted));
    border-radius: 16px;
    color: white;
    border: 1px solid rgba(255, 255, 255, 0.1);
    box-shadow: var(--shadow-lg);
    position: relative;
    overflow: hidden;
  }
  
  .selected-country::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 1px;
    background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
  }
  
  .selected-country h3 {
    margin-bottom: 20px;
    font-size: 1.1rem;
    font-weight: 600;
    opacity: 0.9;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }
  
  .country-details h4 {
    font-size: 1.75rem;
    font-weight: 800;
    margin-bottom: 20px;
    text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  }
  
  .detail-grid {
    display: grid;
    gap: 16px;
  }
  
  .detail-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 12px 16px;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 8px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    transition: all 0.2s ease;
  }
  
  .detail-item:hover {
    background: rgba(255, 255, 255, 0.15);
    transform: translateX(4px);
  }
  
  .detail-label {
    font-weight: 600;
    opacity: 0.9;
    font-size: 0.9rem;
  }
  
  .detail-value {
    font-weight: 700;
    font-size: 1rem;
  }
  
  .instruction {
    margin-top: 24px;
    padding: 24px;
    background: var(--bg-card);
    border: 1px solid var(--border-color);
    border-radius: 16px;
    text-align: center;
    color: var(--text-secondary);
    font-style: italic;
    position: relative;
    overflow: hidden;
  }
  
  .instruction::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 1px;
    background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.1), transparent);
  }
  
  .instruction p {
    font-size: 1rem;
    line-height: 1.6;
  }
</style>
