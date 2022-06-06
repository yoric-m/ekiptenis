<script>
  import Header from "./components/Header.svelte";
  import Comments from "./components/Comments.svelte";
  import TextSize from "./components/TextSize.svelte";
  import Title from "./components/Title.svelte";
  import Body from "./components/Body.svelte";
  import { fetchData } from "./fetchData";

  let article = { title: "", body: "", comments: [[]] };
  let inputSize = 2;
</script>

<div class="main">
  <Header
    on:validUrl={async (url) => (article = await fetchData(url.detail))}
  />
  {#if article.title !== ""}
    <TextSize bind:inputSize />
    <div class="content" style="--fontSizeEm: {inputSize}em">
      <Title title={article.title} />
      <Body body={article.body} />
      <Comments comments={article.comments} />
    </div>
  {/if}
</div>

<style>
  .main {
  }
  .content {
    justify-content: center;
    font-size: var(--fontSizeEm);
    padding: 0px 20px;
  }
</style>
