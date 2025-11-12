<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { geoOrthographic, geoPath, geoGraticule } from 'd3-geo';
  import * as topojson from 'topojson-client';
  
  export let countryData = [];
  export let selectedCountry;
  
  let svg;
  let width = 600;
  let height = 600;
  let rotation = [0, 0];
  let isRotating = true;
  
  const projection = geoOrthographic()
    .scale(250)
    .translate([width / 2, height / 2])
    .clipAngle(90);

  // Function to update projection for responsive sizing
  function updateProjection() {
    if (svg) {
      const rect = svg.getBoundingClientRect();
      width = rect.width || 600;
      height = rect.height || 600;
      
      projection
        .translate([width / 2, height / 2])
        .scale(Math.min(width, height) * 0.4);
    }
  }
  
  const path = geoPath().projection(projection);
  const graticule = geoGraticule();
  
  // Country coordinates for globe rotation
  const countryCoordinates = {
    'US': [-95, 37],
    'CN': [104, 35],
    'IN': [78, 20],
    'BR': [-55, -10],
    'RU': [105, 61],
    'DE': [10, 51],
    'JP': [138, 36],
    'GB': [-3, 55],
    'FR': [2, 46],
    'AU': [133, -27],
    'IT': [12, 42],
    'CA': [-106, 56],
    'KR': [127, 37],
    'ES': [-4, 40],
    'MX': [-102, 23],
    'ID': [113, -0.8],
    'NL': [5, 52],
    'SA': [45, 24],
    'TR': [35, 39],
    'CH': [8, 47],
    'TW': [121, 24],
    'BE': [4, 50],
    'AR': [-64, -34],
    'IE': [-8, 53],
    'IL': [35, 31],
    'TH': [100, 15],
    'NG': [8, 10],
    'EG': [30, 26],
    'ZA': [22, -31],
    'PL': [19, 52],
    'SE': [18, 60],
    'NO': [10, 60],
    'DK': [9, 56],
    'FI': [26, 64],
    'AT': [14, 47],
    'PT': [-8, 40],
    'GR': [22, 39],
    'CZ': [15, 50],
    'HU': [19, 47],
    'RO': [25, 46],
    'HR': [15, 45],
    'SK': [19, 49],
    'SI': [14, 46],
    'EE': [26, 59],
    'LV': [25, 57],
    'LT': [24, 56],
    'LU': [6, 50],
    'IS': [-19, 65],
    'MT': [14, 36],
    'CY': [33, 35]
  };
  
  // Load world map data
  let worldData = null;
  let svgElement = null;
  let colorScale = null;
  let getCountryInfo = null;
  let rotationTimer;
  let isDragging = false;
  let lastMouse = [0, 0];

  // Auto-rotation function
  function startAutoRotation() {
    if (rotationTimer) clearInterval(rotationTimer);
    rotationTimer = setInterval(() => {
      if (isRotating && !isDragging && svgElement) {
        rotation[0] += 0.2; // Slow rotation speed
        projection.rotate(rotation);
        svgElement.selectAll("path").attr("d", path);
      }
    }, 50); // Update every 50ms for smooth rotation
  }
  
  // Function to update country highlighting
  function updateCountryHighlight() {
    if (!svgElement) return;
    
    svgElement.selectAll(".country").each(function(d) {
      const countryInfo = getCountryInfo(d);
      const isSelected = $selectedCountry && countryInfo && countryInfo.iso === $selectedCountry.iso;
      
      d3.select(this)
        .style("fill", d => {
          if (isSelected) return "#ffffff";
          return countryInfo ? colorScale(countryInfo.population) : "#334155";
        })
        .style("stroke", isSelected ? "#e5e7eb" : "#475569")
        .style("stroke-width", isSelected ? 3 : 0.5)
        .style("filter", isSelected ? "brightness(1.3) drop-shadow(0 0 8px rgba(255,255,255,0.8))" : "none");
    });
  }
  
  // Function to rotate globe to show specific country
  function rotateToCountry(countryIso) {
    if (!countryCoordinates[countryIso] || !svgElement) return;
    
    const coords = countryCoordinates[countryIso];
    const targetRotation = [-coords[0], -coords[1]];
    
    // Stop auto-rotation during manual rotation
    isRotating = false;
    
    // Animate rotation to target country
    const startRotation = [...rotation];
    const duration = 1000; // 1 second animation
    const startTime = Date.now();
    
    function animate() {
      const elapsed = Date.now() - startTime;
      const progress = Math.min(elapsed / duration, 1);
      
      // Easing function for smooth animation
      const easeProgress = 1 - Math.pow(1 - progress, 3);
      
      rotation[0] = startRotation[0] + (targetRotation[0] - startRotation[0]) * easeProgress;
      rotation[1] = startRotation[1] + (targetRotation[1] - startRotation[1]) * easeProgress;
      
      projection.rotate(rotation);
      svgElement.selectAll("path").attr("d", path);
      
      if (progress < 1) {
        requestAnimationFrame(animate);
      } else {
        // Update highlighting after rotation completes
        updateCountryHighlight();
        // Resume auto-rotation after a delay
        setTimeout(() => { 
          isRotating = true;
          startAutoRotation();
        }, 3000);
      }
    }
    
    animate();
  }
  
  // Watch for selectedCountry changes
  $: if ($selectedCountry && $selectedCountry.iso) {
    rotateToCountry($selectedCountry.iso);
  }
  
  // Fetch world map data from a public source
  async function loadWorldData() {
    try {
      const response = await fetch('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json');
      const world = await response.json();
      const countries = topojson.feature(world, world.objects.countries);
      return countries;
    } catch (error) {
      console.warn('Failed to load world data, using fallback');
      // Fallback to basic world shapes
      return createFallbackWorldData();
    }
  }
  
  function createFallbackWorldData() {
    return {
      "type": "FeatureCollection",
      "features": [
        {
          "type": "Feature",
          "properties": { "name": "United States", "iso": "US" },
          "geometry": {
            "type": "MultiPolygon",
            "coordinates": [
              [[[-158, 22], [-158, 28], [-154, 28], [-154, 22], [-158, 22]]],
              [[[-125, 49], [-125, 25], [-66, 25], [-66, 49], [-125, 49]]]
            ]
          }
        },
        {
          "type": "Feature", 
          "properties": { "name": "China", "iso": "CN" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[73, 53], [73, 18], [135, 18], [135, 53], [73, 53]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "India", "iso": "IN" },
          "geometry": {
            "type": "Polygon", 
            "coordinates": [[[68, 37], [68, 6], [97, 6], [97, 37], [68, 37]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Brazil", "iso": "BR" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[-74, 5], [-74, -34], [-34, -34], [-34, 5], [-74, 5]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Russia", "iso": "RU" },
          "geometry": {
            "type": "MultiPolygon",
            "coordinates": [
              [[[19, 77], [19, 41], [169, 41], [169, 77], [19, 77]]],
              [[[170, 71], [170, 64], [-170, 64], [-170, 71], [170, 71]]]
            ]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Germany", "iso": "DE" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[5, 55], [5, 47], [15, 47], [15, 55], [5, 55]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Japan", "iso": "JP" },
          "geometry": {
            "type": "MultiPolygon",
            "coordinates": [
              [[[129, 46], [129, 30], [146, 30], [146, 46], [129, 46]]],
              [[[142, 46], [142, 43], [145, 43], [145, 46], [142, 46]]]
            ]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "United Kingdom", "iso": "GB" },
          "geometry": {
            "type": "MultiPolygon",
            "coordinates": [
              [[[-8, 61], [-8, 49], [2, 49], [2, 61], [-8, 61]]],
              [[[-6, 55], [-6, 54], [-5, 54], [-5, 55], [-6, 55]]]
            ]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "France", "iso": "FR" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[-5, 51], [-5, 42], [8, 42], [8, 51], [-5, 51]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Australia", "iso": "AU" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[113, -10], [113, -44], [154, -44], [154, -10], [113, -10]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Italy", "iso": "IT" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[6, 47], [6, 36], [19, 36], [19, 47], [6, 47]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Canada", "iso": "CA" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[-141, 84], [-141, 41], [-52, 41], [-52, 84], [-141, 84]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "South Korea", "iso": "KR" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[124, 39], [124, 33], [132, 33], [132, 39], [124, 39]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Spain", "iso": "ES" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[-10, 44], [-10, 35], [5, 35], [5, 44], [-10, 44]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Mexico", "iso": "MX" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[-118, 33], [-118, 14], [-86, 14], [-86, 33], [-118, 33]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Indonesia", "iso": "ID" },
          "geometry": {
            "type": "MultiPolygon",
            "coordinates": [
              [[[95, 6], [95, -11], [141, -11], [141, 6], [95, 6]]],
              [[[106, -6], [106, -7], [107, -7], [107, -6], [106, -6]]]
            ]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Netherlands", "iso": "NL" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[3, 54], [3, 50], [8, 50], [8, 54], [3, 54]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Saudi Arabia", "iso": "SA" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[34, 33], [34, 16], [56, 16], [56, 33], [34, 33]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Turkey", "iso": "TR" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[26, 42], [26, 35], [45, 35], [45, 42], [26, 42]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Sweden", "iso": "SE" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[11, 69], [11, 55], [24, 55], [24, 69], [11, 69]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Norway", "iso": "NO" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[4, 71], [4, 58], [31, 58], [31, 71], [4, 71]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Denmark", "iso": "DK" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[8, 58], [8, 54], [15, 54], [15, 58], [8, 58]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Finland", "iso": "FI" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[20, 70], [20, 60], [32, 60], [32, 70], [20, 70]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Austria", "iso": "AT" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[9, 49], [9, 46], [17, 46], [17, 49], [9, 49]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Portugal", "iso": "PT" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[-10, 42], [-10, 37], [-6, 37], [-6, 42], [-10, 42]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Greece", "iso": "GR" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[19, 42], [19, 35], [28, 35], [28, 42], [19, 42]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Czech Republic", "iso": "CZ" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[12, 51], [12, 48], [19, 48], [19, 51], [12, 51]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Hungary", "iso": "HU" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[16, 49], [16, 45], [23, 45], [23, 49], [16, 49]]]
          }
        },
        {
          "type": "Feature",
          "properties": { "name": "Romania", "iso": "RO" },
          "geometry": {
            "type": "Polygon",
            "coordinates": [[[20, 48], [20, 43], [30, 43], [30, 48], [20, 48]]]
          }
        }
      ]
    };
  }
  
  onMount(async () => {
    svgElement = d3.select(svg);
    
    // Update projection for current SVG size
    updateProjection();
    
    // Load world data
    worldData = await loadWorldData();
    
    // Create color scale based on population
    const populationExtent = d3.extent(countryData, d => d.population);
    colorScale = d3.scaleSequential()
      .domain(populationExtent)
      .interpolator(t => d3.interpolateRgb("#9ca3af", "#6b7280")(t));
    
    // Draw sphere (ocean)
    svgElement.append("path")
      .datum({ type: "Sphere" })
      .attr("class", "sphere")
      .attr("d", path)
      .style("fill", "url(#oceanGradient)")
      .style("stroke", "#374151")
      .style("stroke-width", 2);
    
    // Add ocean gradient definition
    const defs = svgElement.append("defs");
    const oceanGradient = defs.append("radialGradient")
      .attr("id", "oceanGradient")
      .attr("cx", "30%")
      .attr("cy", "30%");
    
    oceanGradient.append("stop")
      .attr("offset", "0%")
      .style("stop-color", "#4b5563")
      .style("stop-opacity", 0.8);
    
    oceanGradient.append("stop")
      .attr("offset", "70%")
      .style("stop-color", "#374151")
      .style("stop-opacity", 0.9);
    
    oceanGradient.append("stop")
      .attr("offset", "100%")
      .style("stop-color", "#1f2937")
      .style("stop-opacity", 1);
    
    // Draw graticule (grid lines)
    svgElement.append("path")
      .datum(graticule)
      .attr("class", "graticule")
      .attr("d", path)
      .style("fill", "none")
      .style("stroke", "rgba(255,255,255,0.3)")
      .style("stroke-width", 0.5);
    
    // Function to get country info by matching name or ISO
    getCountryInfo = function(feature) {
      // Try to match by ISO code first
      let countryInfo = countryData.find(c => c.iso === feature.properties.iso);
      
      // If no ISO match, try to match by name
      if (!countryInfo) {
        const featureName = feature.properties.name || feature.properties.NAME || feature.properties.NAME_EN;
        countryInfo = countryData.find(c => 
          c.name.toLowerCase() === featureName?.toLowerCase() ||
          featureName?.toLowerCase().includes(c.name.toLowerCase()) ||
          c.name.toLowerCase().includes(featureName?.toLowerCase())
        );
      }
      
      return countryInfo;
    };
    
    // Draw countries
    const countries = svgElement.selectAll(".country")
      .data(worldData.features)
      .enter().append("path")
      .attr("class", "country")
      .attr("d", path)
      .style("fill", d => {
        const countryInfo = getCountryInfo(d);
        return countryInfo ? colorScale(countryInfo.population) : "#334155";
      })
      .style("stroke", "#475569")
      .style("stroke-width", 0.5)
      .style("cursor", "pointer")
      .on("mouseover", function(event, d) {
        d3.select(this)
          .style("fill", "#ffffff")
          .style("stroke", "#e5e7eb")
          .style("stroke-width", 2)
          .style("filter", "brightness(1.2)");
      })
      .on("mouseout", function(event, d) {
        const countryInfo = getCountryInfo(d);
        d3.select(this)
          .style("fill", countryInfo ? colorScale(countryInfo.population) : "#334155")
          .style("stroke", "#475569")
          .style("stroke-width", 0.5)
          .style("filter", "none");
      })
      .on("click", function(event, d) {
        const countryInfo = getCountryInfo(d);
        if (countryInfo) {
          selectedCountry.set(countryInfo);
        }
      });
    
    // Country labels removed for cleaner appearance
    
    // Start auto-rotation
    startAutoRotation();
    
    svgElement
      .on("mousedown", function(event) {
        isDragging = true;
        isRotating = false;
        lastMouse = [event.clientX, event.clientY];
      })
      .on("mousemove", function(event) {
        if (isDragging) {
          const mouse = [event.clientX, event.clientY];
          const deltaX = mouse[0] - lastMouse[0];
          const deltaY = mouse[1] - lastMouse[1];
          
          rotation[0] += deltaX * 0.5;
          rotation[1] -= deltaY * 0.5;
          rotation[1] = Math.max(-90, Math.min(90, rotation[1]));
          
          projection.rotate(rotation);
          svgElement.selectAll("path").attr("d", path);
          
          lastMouse = mouse;
        }
      })
      .on("mouseup", function() {
        isDragging = false;
        setTimeout(() => { 
          isRotating = true;
          startAutoRotation();
        }, 2000);
      });
    
    // Add resize listener to update projection
    const handleResize = () => {
      updateProjection();
      if (svgElement) {
        svgElement.selectAll("path").attr("d", path);
      }
    };
    
    window.addEventListener('resize', handleResize);
    
    return () => {
      clearInterval(rotationTimer);
      window.removeEventListener('resize', handleResize);
    };
  });
</script>

<div class="globe-wrapper">
  <svg bind:this={svg} viewBox="0 0 {width} {height}" preserveAspectRatio="xMidYMid meet"></svg>
  <div class="globe-controls">
    <button on:click={() => { 
      isRotating = !isRotating;
      if (isRotating) startAutoRotation();
    }}>
      {isRotating ? 'Pause Rotation' : 'Resume Rotation'}
    </button>
  </div>
</div>

<style>
  .globe-wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 24px;
    width: 100%;
    min-height: 100%;
  }
  
  svg {
    border-radius: 20px;
    box-shadow: 
      0 25px 50px -12px rgba(0, 0, 0, 0.5),
      0 0 0 1px rgba(255, 255, 255, 0.1);
    background: radial-gradient(circle at 30% 30%, #374151, #1f2937, #000000);
    transition: transform 0.3s ease, box-shadow 0.3s ease;
    width: 100%;
    height: auto;
    max-width: 600px;
    max-height: 600px;
    aspect-ratio: 1;
    display: block;
    margin: 0 auto;
  }
  
  svg:hover {
    transform: scale(1.02);
    box-shadow: 
      0 32px 64px -12px rgba(0, 0, 0, 0.6),
      0 0 0 1px rgba(255, 255, 255, 0.3);
  }
  
  .globe-controls {
    display: flex;
    gap: 16px;
    flex-wrap: wrap;
    justify-content: center;
  }
  
  .globe-controls button {
    background: linear-gradient(135deg, var(--bg-tertiary), var(--text-muted));
    color: white;
    border: 1px solid var(--border-color);
    padding: 14px 28px;
    border-radius: 12px;
    font-weight: 600;
    font-size: 0.95rem;
    cursor: pointer;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    position: relative;
    overflow: hidden;
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.3);
  }
  
  .globe-controls button::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
    transition: left 0.5s ease;
  }
  
  .globe-controls button:hover {
    transform: translateY(-3px) scale(1.05);
    box-shadow: 0 8px 25px rgba(255, 255, 255, 0.2);
    background: linear-gradient(135deg, var(--text-muted), var(--accent-primary));
  }
  
  .globe-controls button:hover::before {
    left: 100%;
  }
  
  .globe-controls button:active {
    transform: translateY(-1px) scale(1.02);
  }
  
  /* Responsive globe sizing */
  @media (max-width: 1024px) {
    .globe-wrapper {
      gap: 20px;
    }
    
    svg {
      max-width: 500px;
      max-height: 500px;
    }
  }
  
  @media (max-width: 768px) {
    .globe-wrapper {
      gap: 16px;
      padding: 0 16px;
    }
    
    svg {
      max-width: 400px;
      max-height: 400px;
      border-radius: 16px;
    }
    
    .globe-controls {
      width: 100%;
      justify-content: center;
    }
    
    .globe-controls button {
      padding: 12px 24px;
      font-size: 0.9rem;
    }
  }
  
  @media (max-width: 480px) {
    .globe-wrapper {
      gap: 12px;
      padding: 0 12px;
    }
    
    svg {
      max-width: 320px;
      max-height: 320px;
      border-radius: 12px;
    }
    
    .globe-controls button {
      padding: 10px 20px;
      font-size: 0.85rem;
      width: 100%;
      max-width: 200px;
    }
  }
</style>
