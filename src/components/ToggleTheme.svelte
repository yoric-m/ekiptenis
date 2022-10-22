<script>
  import { onMount } from "svelte";
  const STORAGE_KEY = "theme";
  const DARK_PREFERENCE = "(prefers-color-scheme: dark)";

  const THEMES = {
    DARK: "dark",
    LIGHT: "light",
  };

  const prefersDarkThemes = () => window.matchMedia(DARK_PREFERENCE).matches;

  const applyTheme = () => {
    document.querySelector(":root").classList.toggle("light");
    if (document.querySelector("#mode").innerHTML === "🌙") {
      document.querySelector("#mode").innerHTML = "🌞";
    } else {
      document.querySelector("#mode").innerHTML = "🌙";
    }
  };

  const toggleTheme = () => {
    const stored = localStorage.getItem(STORAGE_KEY);
    if (stored) {
      localStorage.removeItem(STORAGE_KEY);
    } else {
      localStorage.setItem(
        STORAGE_KEY,
        prefersDarkThemes() ? THEMES.LIGHT : THEMES.DARK
      );
    }
    applyTheme();
  };
  onMount(() => {
    applyTheme();
    window.matchMedia(DARK_PREFERENCE).addEventListener("change", applyTheme);
  });
</script>

<button id="mode" on:click={toggleTheme} />

<style>
  button {
    margin-right: 20px;
    margin-top: 5px;
    background-color: var(--background-color);
    border-width: 0px;
    border-style: none;
    color: var(--text-color);
    cursor: pointer;
    font-size: 1.6em;
    display: inline;
  }
</style>
