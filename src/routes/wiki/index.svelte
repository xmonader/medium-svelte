<script context="module">
	export function preload({ params, query }) {

		console.log("hello.... in index wiki")
	}
</script>


<svelte:head>
	<title>Wiki main</title>

</svelte:head>
<script>

    import { onMount } from 'svelte';
    export let params;
    let wikiarticles = []
	onMount(() => {
        var gun = Gun("ws://127.0.0.1:8000/gun")
        let wikiarticles = gun.get("wiki").get("articles")
        console.log("wikiarticles: " + JSON.stringify(wikiarticles))
	});

</script>


<style>
	ul {
		margin: 0 0 1em 0;
		line-height: 1.5;
	}
</style>

<h1>Recent articles</h1>

<ul>
	{#each wikiarticles as article}
		<!-- we're using the non-standard `rel=prefetch` attribute to
				tell Sapper to load the data for the page as soon as
				the user hovers over the link or taps it, instead of
				waiting for the 'click' event -->
                {article}
		<!-- <li><a rel='prefetch' href='wiki/{article.slug}'>{article.title}</a></li> -->
	{:else}
        no articles yet..
    {/each}
</ul>

