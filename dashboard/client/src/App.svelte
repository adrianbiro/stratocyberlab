<script>
  import Dashboard from './Dashboard.svelte';
  import AssistantLLM from './AssistantLLM.svelte';
  import LoadingOverlay from './LoadingOverlay.svelte';
  import SSH from './Ssh.svelte';
  import { onMount } from 'svelte';
  import ClassDoc from './ClassDoc.svelte';
  import Snowfall from "./Snowfall.svelte";

  let showSSH = false;
  let sshInitialised = false;

  let sshHeight = 50;
  let isSshResizing = false;
  $: dashboardHeight = showSSH ? 100 - sshHeight : 100;

  let widthVertical1st = 66;
  let isVerticalResizing = false;
  $: widthVertical2nd = 98 - widthVertical1st; // 98 is tmp hack to fit in the vertical dragging line

  let chosenChallenge;
  let chosenClass;

  let resizeTerminalContentFunc;

  const snowFlakesEasterEggFeatureFlag = false

  onMount(() => {
    window.addEventListener('mousemove', resizeSSH);
    window.addEventListener('mouseup', stopSshResizing);

    window.addEventListener('mousemove', resizeVertical);
    window.addEventListener('mouseup', stopVerticalResizing);
  });

  function toggleShowSSH() {
    sshInitialised = true;
    showSSH = !showSSH;
  }

  function handleBrandClick() {
    chosenChallenge = undefined;
    chosenClass = undefined;
  }

  function startSshResizing() {
    isSshResizing = true;
  }

  function stopSshResizing() {
    isSshResizing = false;
  }

  function resizeSSH(e) {
    if (!isSshResizing) {
      return;
    }
    const totalHeight = window.innerHeight;
    const newHeight = 100 - (e.clientY / totalHeight) * 100;
    if (newHeight >= 10 && newHeight <= 90) {
      sshHeight = newHeight;
      resizeTerminalContentFunc();
    }
  }

  function startVerticalResizing() {
    isVerticalResizing = true;
  }
  function stopVerticalResizing() {
    isVerticalResizing = false;
  }
  function resizeVertical(e) {
    if (!isVerticalResizing) {
      return;
    }
    const totalWidth = window.innerWidth;
    const newWidth = (e.clientX / totalWidth) * 100;
    if (newWidth >= 30 && newWidth <= 90) {
      widthVertical1st = newWidth;
    }
  }

  let isSnowFalling = snowFlakesEasterEggFeatureFlag;

  function toggleSnowfall() {
    isSnowFalling = !isSnowFalling;
  }


</script>
<style>
  .snowfall-toggle {
    font-size: 1rem;
    margin-right: 15px;
    background-color: transparent; /* Transparent background for cleaner look */
    cursor: pointer; /* Pointer cursor for interactivity */
    transition: all 0.3s ease; /* Smooth transition for hover effect */
  }

  .snowfall-toggle:hover {
    background-color: #f0f0f0; /* Light gray background */
    transform: scale(1.1); /* Slightly enlarge */
  }

  .custom-rounded-button {
    border-radius: 20px 20px 0 0;
  }
</style>

<LoadingOverlay />
{#if snowFlakesEasterEggFeatureFlag && isSnowFalling}
  <Snowfall />
{/if}
<div class="container-fluid p-0 d-flex flex-column" style="height: {dashboardHeight}vh; overflow-y: auto;">
  <!-- Navbar -->
  <nav class="navbar navbar-expand-lg navbar-light bg-light mb-2">
    <div class="container-fluid">
      <a href="/#" class="navbar-brand ms-2" on:click|preventDefault={handleBrandClick}>
        <img src="/media/logo.png" alt="Logo" width="30" height="30" class="d-inline-block align-top" />
        StratoCyberLab
      </a>
    </div>
    {#if snowFlakesEasterEggFeatureFlag}
    <div class="mr-3">
      <button class="btn snowfall-toggle" on:click={toggleSnowfall}>
        ☀️
      </button>
    </div>
    {/if}
  </nav>

  <!-- Main dashboard content -->
  <div class="row flex-grow-1 m-1">
    <div style="width: {widthVertical1st}vw">
      <Dashboard bind:chosenChallenge bind:chosenClass />
    </div>
    <div class="p-0 bg-secondary" on:mousedown={startVerticalResizing} style="width: 3px; cursor: col-resize;"></div>
    <div style="width: {widthVertical2nd}vw">
      {#if chosenClass !== undefined && chosenClass.google_doc_url}
        <ClassDoc docUrl={chosenClass.google_doc_url} />
      {:else}
        <AssistantLLM />
      {/if}
    </div>
  </div>

  <!-- Bottom button to open/close terminal -->
  {#if !showSSH}
    <button class="btn btn-secondary me-auto ms-auto custom-rounded-button" on:click={toggleShowSSH}>
      {#if !sshInitialised}
        Open terminal in the lab ↑
      {:else}
        Reopen terminal ↑
      {/if}
    </button>
  {/if}
</div>

<!-- Terminal -->
<!-- we use this if statement to lazy-initialise the component  -->
<!-- we use it just once in the beginning to keep alive the existing SSH connection  -->
{#if sshInitialised}
  <div class="row flex-grow-1 mx-0 bg-black {!showSSH ? 'visually-hidden' : ''}" style="height: {sshHeight}vh">
    <div class="bg-secondary" on:mousedown={startSshResizing} style="height: 5px; cursor: row-resize;"></div>

    <div class="mx-1 col px-0" style="height: calc(100% - 5px)">
      <SSH bind:resize={resizeTerminalContentFunc} on:hide={toggleShowSSH} />
    </div>
  </div>
{/if}
