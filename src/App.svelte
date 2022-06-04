<script>
  const jsonUrl = "https://dwh.lequipe.fr/api/v2/efr/news/";
  const commentsUrl =
    "https://dwh.lequipe.fr/api/efr/comments/Tennis/Article/XXX/limits/500/lasts/0/best";
  let commUrl = "";
  let url = "";
  let showArticle = false;
  let title = "";
  let body = "";
  let lastFetchUrl = 0;
  let imgEntete = "";
  let arrayComments = [];
  let lastUrl = "";

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
    body = "";
    let arrayComments = [];
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
            title = e.objet.title + " <br/> " + e.objet.long_title;
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
                if (p.__type === "article_paragraph_media") {
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

<div class="content">
  {#if showArticle}
    <h1>{@html title}</h1>
    <div class="body">{@html body}</div>
    <div class="comments">
      {#each arrayComments as oneComment}
        <div class="comment">
          {oneComment}
        </div>
      {/each}
    </div>
    <!-- <textarea bind:value={textArticle} /> -->
    <!-- <div class="article">{textArticle}</div> -->
  {/if}
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
  input::placeholder {
    color: purple;
  }
  .title {
    font-weight: 1.5em;
    font-size: 1.5em;
    color: purple;
    background-color: rgb(211, 246, 246);
    text-align: center;
    display: block;
  }
  .content {
    justify-content: center;
    padding-top: 30px;
  }
  .body {
    margin-top: 30px;
    border: 1px solid black;
    display: block;
  }
  img.center {
    display: block;
    margin: 0 auto;
  }
  .comment {
    border-top: 1px solid black;
    padding: 5px 5px;
  }
</style>
