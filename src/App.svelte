<script>
  const jsonUrl = "https://dwh.lequipe.fr/api/v2/efr/news/";
  let commUrl = "";
  let url = "";
  let showArticle = false;
  let title = "";
  let body = "";
  let lastFetchUrl = 0;
  let imgEntete = "";
  let arrayComments = [];
  let lastUrl = "";
  let inputSize = 2;

  const handleKeyup = () => {
    // if we already handled the request we don't do anything more
    console.log(new Date() - lastFetchUrl);
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
    // Reset the page content before to fetch it
    body = "";
    arrayComments = [];
    lastUrl = url;

    const urlToFetch = jsonUrl + returnMatches[1];
    fetch(urlToFetch)
      .then((response) => {
        return response.json();
      })
      .then((data) => {
        console.log("data");
        console.log(data);
        commUrl = data.comment_count_url.replace(
          "count",
          "limits/500/lasts/0/best"
        );
        goGetComments();
        data.items.map((e) => {
          if (e.layout === "article_feature") {
            title = e.objet.long_title;
            if (e.objet.media.url) {
              console.log("article_feature");
              console.log(e.objet);
              console.log(e.objet.media);
              console.log(e.objet.media.url);
              imgEntete = getImageTag(e.objet.media.url);
            }
          }
          if (e.layout === "article_body") {
            e.objet.paragraphs
              .filter((z) => z.__type != "article_paragraph_pub")
              .filter((p) => p.title != "lire aussi")
              .map((p) => {
                if (p.title) {
                  body += "<h3>" + p.title + "</h3>";
                }
                if (
                  p.__type === "article_paragraph_media" &&
                  p.media.__type !== "base_video"
                ) {
                  console.log("article_paragraph_media");
                  console.log(p);
                  body += getImageTag(p.media.url);

                  // src.replace(`-{width}-{height}-{quality}`, "");
                } else if (
                  p.__type !== "article_paragraph_signature" &&
                  p.content
                ) {
                  body += "<p>" + p.content + "</p><br/>";
                  if (p.layout === "chapo") {
                    body += imgEntete;
                  }
                }
              });
          }
        });
        // textArticle = title;
      });

    //Commentaires

    console.log(urlToFetch);
    // fetch(url):``
  };
  const goGetComments = () => {
    const regArticle = /^http.+\/(.+\/[0-9]{4,})/i;
    const matchesArticle = url.match(regArticle);
    console.log("before");
    console.log(matchesArticle);

    // fetch(commentsUrl.replace("XXX", matchesArticle[1]))
    fetch(commUrl)
      .then((response) => {
        return response.json();
      })
      .then((data) => {
        console.log("Comments");
        console.log(data);
        data.comments.map((comm) => {
          arrayComments = [...arrayComments, comm.text];
          // commentHTML += '<div class="comment">' + comm.text + "</div>";
        });
      });
  };
  const getImageTag = (url) => {
    let src = url;
    // const regImg = /^(http.+)\/\d:\d,\d:\d/i;
    const regImg = /^(http.+[0-9]{6,})\//i;
    const regImh = /quality\}(\/.+)$/i;
    const returnMatches = src.match(regImg);
    const returnMatchet = src.match(regImh);
    if (!returnMatches || !returnMatchet) return "";
    console.log(returnMatches);
    console.log(returnMatchet);
    src = returnMatches[1] + "/203:6,1793:1066-828-552-75" + returnMatchet[1];
    return (
      '<img style="display:block;margin: 0 auto;" src="' + src + '" /><br/>'
    );
  };
</script>

<div class="main">
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

  {#if showArticle}
    <input
      type="range"
      bind:value={inputSize}
      min="1.0"
      max="3.0"
      step="0.01"
    />

    <div class="content" style="--fontSizeEm: {inputSize}em">
      <h1 class="title">{@html title}</h1>
      <div class="body">{@html body}</div>
      <div class="comments">
        {#each arrayComments as oneComment}
          <div class="comment">
            {oneComment}
          </div>
        {/each}
      </div>
    </div>
  {/if}
</div>

<style>
  .main {
    background-color: #ddd;
  }
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
  input[type="range"] {
    width: 100%;
  }
  input[type="text"] {
    margin-top: 10px;
  }
  input::placeholder {
    color: purple;
  }
  .title {
    margin: 5px 5px;
    text-align: center;
  }
  .content {
    justify-content: center;
    font-size: var(--fontSizeEm);
    padding: 0px 20px;
    background-color: #ddd;
  }
  .body {
    border: 1px solid black;
    display: block;
    margin: 20px 11px;
    padding: 0px 20px;
    background-color: white;
  }
  img.center {
    display: block;
    margin: 0 auto;
  }
  .comments {
    border-top: 1px solid black;
  }
  .comment {
    border-top: 1px solid black;
    padding: 5px 5px;
  }
</style>
