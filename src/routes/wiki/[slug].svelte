<script context="module">
  export function preload({ params, query }) {
    let { slug } = params.slug;
    console.log("hello from wiki page " + JSON.stringify(params));
    return { params };
  }
</script>

<script>
  const ENTER_KEY = "Enter";
  const ESCAPE_KEY = "Escape";
  export let params;
  export let currentArticleTitle = "";
  export let currentArticleContent = "";
  export let editingTitle = false;
  export let editingContent = false;
  export let slug = params.slug;
  export let articlebucket;
  export let article = {};
  export let gun;

  var converter = new showdown.Converter();

  $: currentArticleMarkdown = converter.makeHtml(currentArticleContent);
  $: articleSlug = `article://${slug}`;
  function editArticleTitle(article) {
    editingTitle = true;
    currentArticleTitle = article.title;
  }

  function editArticleContent(article) {
    editingContent = true;
    currentArticleContent = article.content;
  }

  function maybeDoneEditTitle(event, article) {
    if (event.key === ENTER_KEY) {
      console.log(`updating title  to ${currentArticleTitle}`);
      editingTitle = false;
      editingContent = false;

      const newArticle = {
        content: currentArticleContent,
        title: currentArticleTitle,
        slug: slug
      };
      gun
        .get("wiki")
        .get("articles")
        .get(articleSlug)
        .put(newArticle);
    }
  }
  // function maybeDoneEditContent(event, article){

  //     if (event.key===ENTER_KEY){
  //         console.log(`updating content  to ${currentArticleContent}`)
  //        editingTitle = false
  //        editingContent = false
  //        const newArticle = {content:currentArticleContent, title:currentArticleTitle}
  //        gun.get("wiki").get("articles").get(slug).put(newArticle)
  //     }
  // }
  function saveContent(event) {
    console.log(`updating content  to ${currentArticleContent}`);
    editingTitle = false;
    editingContent = false;
    const newArticle = {
      content: currentArticleContent,
      title: currentArticleTitle,
      slug: slug
    };
    gun
      .get("wiki")
      .get("articles")
      .get(articleSlug)
      .put(newArticle);
  }

  import { onMount } from "svelte";
  // export let params;

  onMount(() => {
    gun = Gun("ws://127.0.0.1:8000/gun");

    let articlebucket = gun
      .get("wiki")
      .get("articles")
      .get(articleSlug);
    articlebucket.on(data => {
      article = data || {};
      console.log(`setting article to ${JSON.stringify(article)}`);
      currentArticleContent = article.content;
      currentArticleTitle = article.title;
    });

    console.log("article: ", article);
  });
</script>

<svelte:head>
  <title>Wiki main {params.slug}</title>
</svelte:head>
<div>
  <div>
    {#if !article.title || editingTitle}
      <input
        type="text"
        class="form-control form-control-lg"
        bind:value={currentArticleTitle}
        on:keydown={event => maybeDoneEditTitle(event, article)} />
    {:else}
      <div
        class="article-item-title article-item-label content"
        on:dblclick={() => editArticleTitle(article)}>
        <h1>
          <i>{article.title}</i>
        </h1>
        <hr />
      </div>
    {/if}
  </div>

  <div>
    {#if !article.content || editingContent}
      <div class="article-content">
        <button on:click={saveContent}>Save</button>
      </div>
      <div class="markdownpreview">
        {@html currentArticleMarkdown}
      </div>
    {:else}
      <div
        class="article-item-title article-item-label content"
        on:dblclick={() => editArticleContent(article)}>
        {@html currentArticleMarkdown}
      </div>
    {/if}
  </div>

</div>
