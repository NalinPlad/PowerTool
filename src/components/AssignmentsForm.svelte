<script>
  let files;
  let showHelp = false;

  const toggleHelp = () => {
    showHelp = !showHelp;
  };

  $: if (files) {
    // Note that `files` is of type `FileList`, not an Array:
    // https://developer.mozilla.org/en-US/docs/Web/API/FileList
    console.log(files);

    for (const file of files) {
      console.log(`${file.name}: ${file.size} bytes`);
    }
  }
</script>

<div class="">
  <header class="bg-slate-50 p-5 w-screen rounded-lg">
    <div class="text-2xl">
      PowerTool v0.3 by <span class="text-blue-900 tracking-wide">notmysql</span
      >
    </div>

    <div>
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <h1
        on:click={toggleHelp}
        class="my-1 bg-slate-200 w-fit p-1 px-2 hover:bg-slate-300 hover:scale-105 transition-all cursor-default rounded-md italic text-slate-900"
      >
        How to use PowerTool 🤔
      </h1>
      {#if showHelp}
        <p
          class="text-slate-800 bg-amber-100 w-fit p-3 rounded-md before:content-['->'] before:text-slate-500 before:px-3"
        >
          To get your PowerSchool file, log into PowerSchool, right click on the
          page and select <code class="bg-slate-100 p-1 text-sm">Save As</code>
        </p>
      {/if}
    </div>
  </header>

  <main class="m-5">
    <form class="my-5 flex" preventDefault>
      <div class="flex flex-col bg-blue-100 p-3 w-fit rounded-md">
        <label for="pf" class="text-sm m-1 underline">Click here to add your PowerSchool file</label>
        <input
          accept="text/html"
          bind:files
          id="pf"
          name="powerschoolfile"
          type="file"
          class="hidden"
        />
        {#if files}
          {#each Array.from(files) as file}
            <p>{file.name} ({file.size}B)</p>
          {/each}
        {/if}
      </div>

      {#if files}
        <button class="mx-2 bg-green-100 p-3 rounded-md transition-all">😎 Process →</button>
      {/if}
    </form>
  </main>
</div>
