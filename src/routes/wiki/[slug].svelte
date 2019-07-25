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
    let currentArticleTitle = ""
    let currentArticleContent = ""
    let editingTitle = false
    let editingContent = false
    let slug = params.slug
    let articlebucket


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
           articlebucket.get(slug).put({content:currentArticleTitle})

        }
    }
    function maybeDoneEditContent(event, article){
        
        if (event.key===ENTER_KEY){
            console.log(`updating content  to ${currentArticleContent}`)
            editingContent = false
            articlebucket.get(slug).put({content:currentArticleContent})

        }
    }

    import { onMount } from 'svelte';
    // export let params;
    let article;


	onMount(() => {
        var gun = Gun("ws://127.0.0.1:8000/gun")

        let articlebucket = gun.get("wiki").get("articles").get(slug)
        articlebucket.on( (data) => article = data)
        console.log("article: ", article)
	});

    // console.log("params: " + JSON.stringify(params))
    // console.log("slug: " + slug)

    console.log(article)
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
        {#if !article || editingTitle}
            <label>Title:</label><input type="text" class="title-input" bind:value="{currentArticleTitle}" on:keydown={(event) => maybeDoneEditTitle(event, article)} />
        {:else}
            <div class="article-item-title article-item-label" on:dblclick={() => editArticleTitle(article)}>
                {article.title}
            </div>
        {/if}
    </div>

    <div>
        {#if !article || editingContent}
            <label>content:</label><textarea class="content-input" bind:value="{currentArticleContent}" on:keydown={(event) => maybeDoneEditContent(event, article)} />
        {:else}
            <div class="article-item-title article-item-label" on:dblclick={() => editArticleContent(article)}>
                {article.title}
            </div>
        {/if}
    </div>

</div>