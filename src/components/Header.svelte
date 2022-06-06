<script>
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();

  export let url = "";
  let lastFetchUrl = 0;
  let showArticle = false;
  let lastUrl = "";

  const handleKeyup = () => {
    // if we already handled the request we don't do anything more
    if (showArticle && new Date() - lastFetchUrl < 100) return;
    if (url === lastUrl) return;

    // regex with http.../234235523
    const regexUrl = /^http.+\/(\d{4,})$/i;
    const returnMatches = url.match(regexUrl);
    showArticle = returnMatches !== null;

    // No matches we exit early
    if (!showArticle) return;

    // Record the time we fetched a valid url
    lastFetchUrl = new Date();
    lastUrl = url;
    dispatch("validUrl", url);
  };
</script>

<div class="header">
  <h1>EkipTenis</h1>
  <input
    type="text"
    on:keyup={handleKeyup}
    on:click={handleKeyup}
    bind:value={url}
    placeholder="Enter a url..."
  />
</div>

<style>
  h1 {
    color: purple;
    padding-right: 40px;
  }
  .header {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 50px;
    background-color: lightgrey;
  }
  input {
    width: 1000px;
    padding: 5px 10px;
    font-size: 1.3em;
    background-color: lightgrey;
  }
  input[type="text"] {
    margin-top: 10px;
  }
  input::placeholder {
    color: purple;
  }
</style>
