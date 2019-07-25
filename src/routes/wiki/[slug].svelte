<script context="module">
	export function preload({ params, query }) {
            let {slug} = params.slug
            console.log("hello from wiki page " + JSON.stringify(params))
            return {params}
    }
</script>


<svelte:head>
	<title>Wiki main {params.slug}</title>
</svelte:head>


<script>
    const ENTER_KEY = "Enter"
    const ESCAPE_KEY = "Escape"
    export let params
    export let currentArticleTitle = ""
    export let currentArticleContent = ""
    export let editingTitle = false
    export let editingContent = false
    export let slug = params.slug
    export let articlebucket
    export let article = {}
    export let gun

    var converter = new showdown.Converter()

    $: currentArticleMarkdown =  converter.makeHtml(currentArticleContent);

    function editArticleTitle(article){
        editingTitle = true
        currentArticleTitle = article.title
    }

    function editArticleContent(article){
        editingContent = true
        currentArticleContent = article.content
    }

    function maybeDoneEditTitle(event, article){
        
        if (event.key===ENTER_KEY){
            console.log(`updating title  to ${currentArticleTitle}`)
           editingTitle = false
           editingContent = false

           const newArticle = {content:currentArticleContent, title:currentArticleTitle}
           gun.get("wiki").get("articles").get(slug).put(newArticle)

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
    function saveContent(event){
        
            console.log(`updating content  to ${currentArticleContent}`)
           editingTitle = false
           editingContent = false
           const newArticle = {content:currentArticleContent, title:currentArticleTitle}
           gun.get("wiki").get("articles").get(slug).put(newArticle)
    }

    import { onMount } from 'svelte';
    // export let params;


	onMount(() => {
        gun = Gun("ws://127.0.0.1:8000/gun")

        let articlebucket = gun.get("wiki").get("articles").get(slug)
        articlebucket.on( (data) => {
            article = data || {};
             console.log(`setting article to ${JSON.stringify(article)}`)
             currentArticleContent = article.content
             currentArticleTitle = article.title
        })

        console.log("article: ", article)
	});

</script>




<style>
	/*
		By default, CSS is locally scoped to the component,
		and any unused styles are dead-code-eliminated.
		In this page, Svelte can't know which elements are
		going to appear inside the {{{post.html}}} block,
		so we have to use the :global(...) modifier to target
		all elements inside .content
	*/
	.content :global(h2) {
		font-size: 1.4em;
		font-weight: 500;
	}

	.content :global(pre) {
		background-color: #f9f9f9;
		box-shadow: inset 1px 1px 5px rgba(0,0,0,0.05);
		padding: 0.5em;
		border-radius: 2px;
		overflow-x: auto;
	}

	.content :global(pre) :global(code) {
		background-color: transparent;
		padding: 0;
	}

	.content :global(ul) {
		line-height: 1.5;
	}

	.content :global(li) {
		margin: 0 0 0.5em 0;
	}
</style>


<div>
    <div>
        {#if !article.title || editingTitle}
            <label>Title:</label><input type="text" class="title-input" bind:value="{currentArticleTitle}" on:keydown={(event) => maybeDoneEditTitle(event, article)} />
        {:else}
            <div class="article-item-title article-item-label content" on:dblclick={() => editArticleTitle(article)}>
                {article.title}
            </div>
        {/if}
    </div>

    <div>
        {#if !article.content || editingContent}
            <label>content:</label><textarea class="content-input" bind:value="{currentArticleContent}" />
            <div class="markdownpreview">
                {@html currentArticleMarkdown}
            </div>
            <button on:click={saveContent}>Save</button>
        {:else}
            <div class="article-item-title article-item-label content" on:dblclick={() => editArticleContent(article)}>
                {@html currentArticleMarkdown}
            </div>

        {/if}
    </div>


</div>