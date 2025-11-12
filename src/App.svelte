<script>
  import Globe from './components/Globe.svelte';
  import StatsPanel from './components/StatsPanel.svelte';
  import { countryData } from './data/countryData.js';
  import { writable } from 'svelte/store';

  export let selectedCountry = writable(null);
  
  $: totalPopulation = countryData.reduce((sum, country) => sum + country.population, 0);
  $: totalGDP = countryData.reduce((sum, country) => sum + country.gdp, 0);
  $: averageLifeExpectancy = countryData.reduce((sum, country) => sum + country.lifeExpectancy, 0) / countryData.length;
</script>

<div class="dashboard">
  <header class="header">
    <h1>Earth Statistics</h1>
    <p>Explore interactive global data through a rotating 3D globe. Click countries to discover detailed statistics, compare populations, economies, and quality of life indicators across nations worldwide.</p>
  </header>
  
  <div class="globe-container">
    <Globe {countryData} {selectedCountry} />
  </div>
  
  <div class="stats-panel">
    <StatsPanel 
      {countryData} 
      {selectedCountry}
      {totalPopulation}
      {totalGDP}
      {averageLifeExpectancy}
    />
  </div>
</div>
