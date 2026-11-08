<script>
	import { setContext } from "svelte";
	import { browser } from "$app/environment";
	import Index from "$components/Index.svelte";
	import Questions from "$components/Questions.svelte";
	import copy from "$data/copy.json";

	let { data } = $props();

	const { title, description, url, keywords } = copy;
	setContext("copy", copy);
	setContext("data", data);
</script>

<!-- <Index />
<QuestionsIntro></QuestionsIntro> -->
<Questions></Questions>
<!-- <Connections></Connections> -->