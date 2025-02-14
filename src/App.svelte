<script>
  import searchResults from './tmp.json';
  import Toggle from './Toggle.svelte';
  import { onMount } from 'svelte';
  import { onDestroy } from 'svelte';
  import overviews from "./overviews.json";
  import interleavedResults from "./interleaved.json";
  

  let viewMode = "default";
  let results = searchResults;
  let categories = {};
  
  let showMore = true;
  let highlightMedication = true;
  let highlightSymptom = true;
  let highlightSpecialist = true;
  let highlightCare = true;

  function filterResults() {
  const query = document.getElementById('search-bar').value.toLowerCase();
  const resultContainers = document.querySelectorAll('.result, .result-row');

  resultContainers.forEach(result => {
    const snippetElement = result.querySelector('.snippet');
    const originalSnippet = snippetElement.getAttribute('data-original-snippet') || snippetElement.innerHTML;

    if (!snippetElement.hasAttribute('data-original-snippet')) {
      snippetElement.setAttribute('data-original-snippet', originalSnippet); // Store the original content
    }

    if (query) {
      if (originalSnippet.toLowerCase().includes(query)) {
        result.style.display = 'block';
        snippetElement.innerHTML = highlightText(originalSnippet, query); // Apply highlighting
      } else {
        result.style.display = 'none';
      }
    } else {
      // Reset to original content when query is empty
      result.style.display = 'block';
      snippetElement.innerHTML = originalSnippet;
    }
  });
}

function highlightText(text, query) {
  if (!query) return text;

  const parts = text.split(/(<[^>]+>)/g); // Split into HTML tags and text
  return parts
    .map(part => {
      if (part.startsWith('<')) return part; // Leave HTML tags untouched
      const regex = new RegExp(`(${query.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')})`, 'gi');
      return part.replace(regex, '<span class="search-highlight">$1</span>');
    })
    .join('');
}



  function switchView(mode) {
    viewMode = mode;
    if (mode === "interleaved") {
      organizeByCategory(interleavedResults);
    }
  }

  function organizeByCategory(data) {
  categories = {};
  Object.entries(data).forEach(([category, entries]) => {
    categories[category] = Object.entries(entries).map(([url, snippet]) => ({
      url,
      snippet,
      originalSnippet: snippet, // Store the original snippet for filtering
      page: new URL(url).hostname,
    }));
  });
}

function filterInterleavedResults() {
  const query = document.getElementById('search-bar').value.toLowerCase();

  Object.keys(categories).forEach(category => {
    categories[category] = categories[category].map(entry => {
      const matches = entry.originalSnippet.toLowerCase().includes(query);
      
      // If there's a search query, highlight it first
      let processedSnippet = matches && query 
        ? highlightText(entry.originalSnippet, query)
        : entry.originalSnippet;
        
      entry.snippet = processedSnippet;
      entry.hidden = !matches && query.length > 0; // Only hide if there's a query and no match
      return entry;
    });
  });

  // Force a reactive update
  categories = {...categories};
}



  onMount(() => {
    organizeByCategory(interleavedResults);
  });

  onDestroy(() => {
    console.log('Component is being destroyed');
  });
</script>


<style>
/* Container for the entire layout */
.container {
  display: flex;
  position: relative;
  gap: 20px;
  padding-left: 20px;
}

/* Fixed left panel */
.left-panel {
  position: fixed;
  /* top: 40px; */
  left: 20px;
  right:20px;
  width: 200px;
  height: calc(100vh - 40px);
  /* padding-right: 20px; */
  border-right: 1px solid #ccc;
  overflow-y: auto;
}

/* Main content area with offset to account for fixed left panel */
/* Main content area with offset to account for fixed left panel */
.main-content {
  margin-left: 190px;
  flex: 1;
  width: 1000px; /* Set a minimum width to prevent shrinking */
}

/* Maintain other styles */
.toggle-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.search-bar {
  width: 100%;
  padding: 10px;
  margin-bottom: 20px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.search-results {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  /* width: 1200px; */
}
/* 
.redundant {
  color: gray;
  opacity: 0.7;
} */
:global(.rare) {  @apply font-bold;}


.search-results-interleaved {
  display: grid;
  gap: 20px;
  /* width: 1200px; */
  margin: 0 auto;
}

.result {
  margin-bottom: 20px;
}

.show-more-btn {
    width: 100%; /* Full width */
    border: 2px solid #4A90E2; /* Blue border */
    background: transparent; /* Transparent background */
    color: #000; /* Black text */
    font-size: 15px;
    padding: 10px 0;
    border-radius: 25px; /* Rounded edges */
    cursor: pointer;
    text-align: center;
    font-weight: 400;
    transition: background 0.3s ease-in-out, color 0.3s ease-in-out;
  }

  .show-more-btn:hover {
    background: #d3e2fe; /* Light blue hover effect */
  }

  .arrow {
    color: #4A90E2; /* Blue arrow */
    font-size: 14px;
    margin-left: 5px;
  }



.page-link {
    flex: 1;
    text-align: right;
    /* margin-left: 10px; */
    color: #1a0dab;
    text-decoration: none;
  }

  .page-link:hover {
    text-decoration: underline;
  }

  .domain {
    color: #006621;
    font-size: 14px;
    margin-bottom: 4px;
  }

  .snippet {
    font-size: 14px;
    line-height: 1.4;
    flex: 5;
  }

  button {
    width: 130px
  }

/* Dropdown styles */
.dropdown {
  position: relative;
  display: inline-block;
  margin-bottom: 20px;
  width: 130px;
  flex-shrink: 0;
}

.dropdown-content {
  display: none;
  position: absolute;
  background-color: #f9f9f9;
  width: 130px;
  box-shadow: 0px 8px 16px rgba(0, 0, 0, 0.2);
  z-index: 1;
}


.dropdown:hover .dropdown-content {
  display: block;
}
  
.result-row {
    display: flex;
    justify-content: space-between;
    margin-bottom: 2px;
  }

  :global(.medication) { background-color: transparent; }
  :global(.symptom) { background-color: transparent; }
  :global(.specialist) { background-color: transparent; }
  :global(.care) { background-color: transparent; }

  :global(.highlight-medication .medication) { @apply bg-red-100 rounded-md hover:bg-red-200 hover:font-semibold;}
  :global(.highlight-symptom .symptom) { @apply bg-orange-100 rounded-md hover:bg-orange-200 hover:font-semibold;}
  :global(.highlight-specialist .specialist) { @apply bg-yellow-100 rounded-md hover:bg-yellow-200 hover:font-semibold;}
  :global(.highlight-care .care) { @apply bg-green-100 rounded-md hover:bg-green-200 hover:font-semibold;}

  :global(.search-highlight) {
    @apply font-bold underline decoration-sky-500 decoration-2;
    /* cursor: pointer; */
  }

</style>


<div class="container">
  <div class="left-panel">
    
  
      <!-- Dropdown for View Selection -->
      
      <div class="dropdown">
        <!-- <h4 class="h4">View Mode</h4> -->
        <!-- <button id="multiLevelDropdownButton" data-dropdown-toggle="multi-dropdown" 
 -->
        <button class="text-white bg-blue-500 hover:bg-blue-600 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-xs px-5 py-2.5 text-center inline-flex items-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800" type="button"> {viewMode === "default" ? "Entries" : "Sentences"}<svg class="w-2.5 h-2.5 ms-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 10 6"><path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m1 1 4 4 4-4"/>
        </svg></button>
        <div class="dropdown-content text-xs ">
          <button on:click={() => switchView("default")} class='text-left text-xs  bg-white '>Entries</button>
          <button on:click={() => switchView("interleaved")} class='text-left text-xs bg-white '>Sentences</button>
        </div>
      </div>
  
    <br>
    <div class="toggle-container">
      <h4 class="h4">Key</h4>
      <Toggle bind:checked={highlightMedication} label="Medication" active="bg-red-200" />
      <Toggle bind:checked={highlightSymptom} label="Symptom" active="bg-orange-200" />
      <Toggle bind:checked={highlightSpecialist} label="Specialist" active="bg-yellow-200" />
      <Toggle bind:checked={highlightCare} label="Care" active="bg-green-200" />
    </div>
  </div>
  <div class="main-content text">
    
    {#if viewMode === "default"}
      <div>
        <input type="text" id="search-bar" class="search-bar" placeholder="Search..." on:input={filterResults}>
      </div>

      <div class:highlight-medication={highlightMedication} class:highlight-symptom={highlightSymptom} class:highlight-specialist={highlightSpecialist} class:highlight-care={highlightCare}>

      <div class="text-[15px]">

        <div class="flex items-center gap-2 mb-4">
          <svg class="w-6 h-6 text-blue-600" viewBox="0 0 471 471" fill="currentColor">
            <path d="M235.5 471C235.5 438.423 229.22 407.807 216.66 379.155C204.492 350.503 187.811 325.579 166.616 304.384C145.421 283.189 120.498 266.508 91.845 254.34C63.1925 241.78 32.5775 235.5 0 235.5C32.5775 235.5 63.1925 229.416 91.845 217.249C120.498 204.689 145.421 187.811 166.616 166.616C187.811 145.421 204.492 120.497 216.66 91.845C229.22 63.1925 235.5 32.5775 235.5 0C235.5 32.5775 241.584 63.1925 253.751 91.845C266.311 120.497 283.189 145.421 304.384 166.616C325.579 187.811 350.503 204.689 379.155 217.249C407.807 229.416 438.423 235.5 471 235.5C438.423 235.5 407.807 241.78 379.155 254.34C350.503 266.508 325.579 283.189 304.384 304.384C283.189 325.579 266.311 350.503 253.751 379.155C241.584 407.807 235.5 438.423 235.5 471Z"></path>
          </svg>
          <h2 class="text-xl font-semibold">AI Overview</h2>
        </div>
        <!-- <h2 class="text-xl font-semibold">AI Overview</h2> -->
        <p>Endometriosis can be managed with medication and lifestyle changes.</p>
        
        <!-- Hidden content (expands when button is clicked) -->
        {#if showMore}
          <div class="grid grid-cols-2 gap-4 mt-4">
            {#each Object.entries(overviews) as [key, value]}
            <blockquote class="dark:text-white p-4 bg-[#f1f5ff] rounded-lg text-[14px] card-hover">
                <div class="flex items-center gap-2 mb-2">
                  <h3 class="text-lg font-semibold capitalize">{key}</h3>
                </div>
                <span class="inline text-[15px]">{@html value}</span>
              </blockquote>
            {/each}
          </div>
        {/if}
        <!-- "Show More" button with transparent background and blue outline -->
        <button 
          class="show-more-btn mt-4"
          on:click={() => showMore = !showMore}>
          {showMore ? "Show Less" : "Show More"} <span class="arrow">{showMore ? "▲" : "▼"}</span>
        </button>      
  
      </div>
      <hr class="mt-4">
      <br>

      <div class="search-results">
        {#each searchResults as result}
          <div class="result">
            <a href={result.url} class="page-link" target="_blank">{result.page}</a>
            <div class="domain">{result.domain}</div>
            <div class="snippet text-xs">{@html result.annotated_snippet}</div>
          </div>
        {/each}
      </div>
    </div>
    {/if}
 
  
    {#if viewMode === "interleaved"}
  <div>
    <input
      type="text"
      id="search-bar"
      class="search-bar"
      placeholder="Search..."
      on:input={() => filterInterleavedResults()}
    />
  </div>

  <div class="search-results-interleaved" 
       class:highlight-medication={highlightMedication} 
       class:highlight-symptom={highlightSymptom} 
       class:highlight-specialist={highlightSpecialist} 
       class:highlight-care={highlightCare}>
    {#each Object.entries(categories) as [category, entries]}
      {@const visibleEntries = entries.filter(entry => !entry.hidden)}
      {#if visibleEntries.length > 0}
      {#if 
        (category === 'Medication' && highlightMedication) || 
        (category === 'Symptom' && highlightSymptom) || 
        (category === 'Specialist' && highlightSpecialist) || 
        (category === 'Care' && highlightCare)}
        <div class="category-section">
          <h3 class="h3 capitalize font-semibold">{category}</h3>
          {#if overviews[category]}
          <!-- <blockquote class="text-small italic font-semithin text-gray-900 dark:text-white">
            <p class="mb-4 text-gray-600">✨ AI Overview: {overviews[category]}</p>
          </blockquote> -->




            <blockquote class="dark:text-white p-4 bg-[#f1f5ff] rounded-lg mb-4 mt-4 card-hover">
              <div class="flex items-center gap-2 mb-4">
                <svg class="w-6 h-6 text-blue-600" viewBox="0 0 471 471" fill="currentColor">
                  <path d="M235.5 471C235.5 438.423 229.22 407.807 216.66 379.155C204.492 350.503 187.811 325.579 166.616 304.384C145.421 283.189 120.498 266.508 91.845 254.34C63.1925 241.78 32.5775 235.5 0 235.5C32.5775 235.5 63.1925 229.416 91.845 217.249C120.498 204.689 145.421 187.811 166.616 166.616C187.811 145.421 204.492 120.497 216.66 91.845C229.22 63.1925 235.5 32.5775 235.5 0C235.5 32.5775 241.584 63.1925 253.751 91.845C266.311 120.497 283.189 145.421 304.384 166.616C325.579 187.811 350.503 204.689 379.155 217.249C407.807 229.416 438.423 235.5 471 235.5C438.423 235.5 407.807 241.78 379.155 254.34C350.503 266.508 325.579 283.189 304.384 304.384C283.189 325.579 266.311 350.503 253.751 379.155C241.584 407.807 235.5 438.423 235.5 471Z"></path>
                </svg>
                <h2 class="text-large font-semibold">AI Overview</h2>
              </div>
                <span>{@html overviews[category]}</span> 

            </blockquote>
          {/if}

          {#each visibleEntries as entry (entry.url)}
            <div class="result-row">
              <div class="snippet">{@html entry.snippet}</div>
              <a 
              href={entry.url.replace(/_[^_]*$/, '')} 
              class="page-link" 
              title={entry.page}
            >
              {entry.page.length > 15 ? entry.page.slice(0, 15) + '...' : entry.page}
            </a>
            </div>
          {/each}
        </div>
        {/if}
      {/if}
    {/each}
  </div>
{/if}
  </div>
  
</div>