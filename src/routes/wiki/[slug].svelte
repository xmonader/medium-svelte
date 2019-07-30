<script context="module">
  export function preload({ params, query }) {
    let { slug } = params.slug;
    console.log("hello from wiki page " + JSON.stringify(params));
    return { params };
  }
</script>

<script>
  import { onMount } from "svelte";

  const ENTER_KEY = "Enter";
  const ESCAPE_KEY = "Escape";
  export let params;
  export let currentArticleTitle = "";
  export let currentArticleContent = "";
  export let editingTitle = false;
  export let editingContent = false;
  export let slug = params.slug;
  export let articlebucket = {};
  export let article = {};
  export let currentTags = [];
  export var gun;

  var converter = new showdown.Converter();

  $: currentArticleMarkdown = converter.makeHtml(currentArticleContent);
  $: articleContentMarkdown = converter.makeHtml(article.content);

  let tagsSet = () =>
    Array.from(new Set(currentTags.filter(t => t !== undefined && t.slug)));

  onMount(() => {
    gun = Gun("ws://127.0.0.1:8000/gun");
    // gun = Gun();
    articlebucket = gun.get("wiki://1").get(slug);

    articlebucket.get("list_tags").map((tag, i) => {
      if (!tag) {
        return;
      }
      console.log("init adding tag.." + tag.tag);
      console.log(`init currentTags: ${currentTags}`);
      console.log(
        `${currentTags} includes ${tag.tag}`,
        currentTags.includes(tag.tag)
      );
      if (tag !== undefined && !currentTags.includes(tag.tag)) {
        console.log(`current tags in init: ${currentTags} and tag: ${tag.tag}`);
        currentTags.push(tag.tag);
      }
      currentTags = currentTags;
    });
    articlebucket.on(data => {
      article = data || {};
      console.log(`setting article to ${JSON.stringify(article)}`);
      currentArticleContent = article.content;
      currentArticleTitle = article.title;
    });

    console.log("article: ", article);
  });

  function saveArticle(slug, theArticle) {
    gun
      .get("wiki://1")
      .get(slug)
      .put(theArticle);
  }
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
      //   let articleTags = articlebucket.get("tags");

      articlebucket.put({
        content: currentArticleContent,
        title: currentArticleTitle,
        slug: slug
        // articleTags: tags
      });
    }
  }

  function saveContent(event) {
    console.log(`updating content  to ${currentArticleContent}`);

    editingTitle = false;
    editingContent = false;
    // let articleTags = articlebucket.get("tags");

    articlebucket.put({
      content: currentArticleContent,
      title: currentArticleTitle,
      slug: slug
    });
  }
  function addTag(input) {
    let newTag = input.value;
    input.value = "";
    console.log("adding tag.." + newTag);
    console.log(`currentTags: ${currentTags}`);
    console.log(
      `${currentTags} includes ${newTag}`,
      currentTags.includes(newTag)
    );
    if (currentTags.includes(newTag)) {
      console.log(`tag ${newTag} exists already`);
      return;
    }
    console.log("continuing");
    let t = gun.get("wiki://1").get(newTag + "_tag");
    t.put({ tag: newTag });
    articlebucket.get("list_tags").set(t);
    currentTags = [...currentTags, newTag];
  }
  function removeTag(idx) {
    console.log("removing tag.." + idx + currentTags[idx]);
    console.log(`currentTags: ${currentTags}`);

    let theTag = currentTags[idx];
    let t = gun.get("wiki://1").get(theTag + "_tag");
    currentTags = [...currentTags.slice(0, idx), ...currentTags.slice(idx + 1)];
    articlebucket.get("list_tags").unset(t);
  }

  function enter(node, callback) {
    function onkeydown(event) {
      if (event.which === 13) callback(node);
    }
    node.addEventListener("keydown", onkeydown);
    return {
      destroy() {
        node.removeEventListener("keydown", onkeydown);
      }
    };
  }
</script>

<svelte:head>
  <title>Wiki main {params.slug}</title>
</svelte:head>
<div>
  <div>
    <div class="tag-list">
      {#each tagsSet() as tag, idx}
        {#if tag !== undefined}
          <span class="tag-default tag-pill" on:click={() => removeTag(idx)}>
            <i class="ion-close-round" />
            {tag}
          </span>
        {/if}
      {/each}
    </div>

    <input
      class="form-control"
      type="text"
      placeholder="Enter tags"
      use:enter={addTag} />
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
        <textarea
          rows="20"
          class="form-control form-control-lg"
          bind:value={currentArticleContent} />
        <button
          class="btn btn-lg pull-xs-right btn-primary"
          on:click={saveContent}>
          Save
        </button>
      </div>
      <div class="markdownpreview">
        {@html currentArticleMarkdown}
      </div>
    {:else}
      <div
        class="article-item-title article-item-label content"
        on:dblclick={() => editArticleContent(article)}>
        {@html articleContentMarkdown}
      </div>
    {/if}

  </div>

</div>
