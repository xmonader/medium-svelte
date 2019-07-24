<script context="module">
	// the (optional) preload function takes a
	// `{ path, params, query }` object and turns it into
	// the data we need to render the page
	export async function preload(page, session) {
		// the `slug` parameter is available because this file
		// is called [slug].svelte
		const { slug } = page.params;
        console.log(`${slug``)
		// `this.fetch` is a wrapper around `fetch` that allows
		// you to make credentialled requests on both
		// server and client
		const data = { title:"yaa", name:slug, favnum:5 }
		return data;
	}
</script>


<script>
	export let data;
</script>

<svelte:head>
	<title>{data.title}</title>
</svelte:head>

<h1>{data.name}</h1>

{data.favnum}