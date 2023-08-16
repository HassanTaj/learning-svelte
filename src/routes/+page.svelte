<script>
	import Counter from './Counter.svelte';
	import welcome from '$lib/images/svelte-welcome.webp';
	import welcome_fallback from '$lib/images/svelte-welcome.png';
	export class Home {
		static get Welcome() { return 'Welcome';}
	}
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>

<div class="card">
	<div class="card-body">
		<h4 class="card-title">Title</h4>
		<p class="card-text">{Home.Welcome}</p>
	</div>
</div>

<div class="flex shadow-neutral-500 text-green-200 p-4 border-2 bg-slate-500 rounded my-3">
	<p class="font-bold"> WTF IS THIS</p>
</div>

