<script context="module">
	export function preload({ params, query }) {

	}
</script>


<svelte:head>
	<title>Wiki main</title>

</svelte:head>
<script>

    import { onMount } from 'svelte';
    export let params;
    export var gun;

    function getArticles() {
        let res = []
        let gun = Gun("ws://127.0.0.1:8000/gun")

        gun.get("wiki").get("articles").map().on( (v, k) => res.push({slug: v.slug, title: v.title, content:v.content}))
        return res
    }


</script>


<style>
	ul {
		margin: 0 0 1em 0;
		line-height: 1.5;
	}
</style>

<h1>Recent articles</h1>

<ul>
	{#each getArticles() as article}
		<!-- we're using the non-standard `rel=prefetch` attribute to
				tell Sapper to load the data for the page as soon as
				the user hovers over the link or taps it, instead of
				waiting for the 'click' event -->
		<li><a rel='prefetch' href='wiki/{article.slug}'>{article.title}</a></li>
	{:else}
        no articles yet..
    {/each}
</ul>

