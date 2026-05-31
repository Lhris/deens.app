<script lang="ts">
	import { fly } from 'svelte/transition';
	import { Button } from '$lib/components/ui/button/index.js';
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link href="https://fonts.googleapis.com/css2?family=Syne:wght@800&display=swap" rel="stylesheet" />
	<title>About · deens</title>
</svelte:head>

<div class="min-h-screen bg-stone-50">
	<!-- Navbar -->
	<nav class="sticky top-4 z-20 mx-auto flex w-4/5 items-center justify-between rounded-full border border-white/50 bg-white/70 px-8 py-2.5 shadow-sm backdrop-blur-xl">
		<a href="/" style="font-family: 'Syne', sans-serif;" class="text-sm font-extrabold uppercase tracking-tight text-stone-900">deens</a>
		<ul class="flex list-none items-center gap-8">
			<li><a href="/" class="text-[10px] uppercase tracking-widest text-stone-400 transition-colors hover:text-stone-900">Home</a></li>
			<li><a href="/library" class="text-[10px] uppercase tracking-widest text-stone-400 transition-colors hover:text-stone-900">Library</a></li>
			<li><a href="/community" class="text-[10px] uppercase tracking-widest text-stone-400 transition-colors hover:text-stone-900">Community</a></li>
		</ul>
		<Button size="sm" class="rounded-full px-4 text-[9px] tracking-wide">login</Button>
	</nav>

	<main class="mx-auto max-w-2xl px-6 pb-24 pt-20">
		<!-- Header -->
		<div in:fly={{ y: 12, duration: 400 }}>
			<p class="mb-1.5 text-[9px] uppercase tracking-[0.35em] text-stone-400">The project</p>
			<h1 style="font-family: 'Syne', sans-serif;" class="mb-12 text-4xl font-extrabold uppercase tracking-tight text-stone-900">About</h1>
		</div>

		<!-- Body -->
		<div class="space-y-10 text-sm leading-7 text-stone-600">
			<div in:fly={{ y: 12, duration: 400, delay: 60 }}>
				<p class="mb-3 text-[9px] uppercase tracking-widest text-stone-400">What is deens</p>
				<p>
					Deens is a community library for tinned fish. It exists because good tinned fish is genuinely interesting — and finding, comparing, and remembering it shouldn't be hard. We catalogue tins, track nutrition and ingredients, and let people log what they've tried.
				</p>
			</div>

			<div in:fly={{ y: 12, duration: 400, delay: 120 }}>
				<p class="mb-3 text-[9px] uppercase tracking-widest text-stone-400">Why tinned fish</p>
				<p>
					Tinned fish has had a quiet renaissance. What was once pantry filler is now a serious category: small producers, single-origin catches, traditional preservation methods. A good tin of sardines rivals a good piece of fresh fish — it's just less perishable and more portable.
				</p>
				<p class="mt-4">
					The problem is that the market is opaque. Labels vary wildly. Brands you've never heard of are often better than the ones you have. Deens tries to bring some order to that.
				</p>
			</div>

			<div in:fly={{ y: 12, duration: 400, delay: 180 }}>
				<p class="mb-3 text-[9px] uppercase tracking-widest text-stone-400">Built by</p>
				<p>
					Deens is a side project by
					<a
						href="https://github.com/lhris"
						target="_blank"
						rel="noopener noreferrer"
						class="font-medium text-stone-800 underline underline-offset-2 transition-colors hover:text-stone-500"
					>
						lhris
					</a>
					— a software engineer who eats too many sardines.
				</p>
			</div>

			<div in:fly={{ y: 12, duration: 400, delay: 240 }} class="flex gap-3 pt-2">
				<Button href="/library" class="rounded-full px-5 text-[9px] tracking-wide">Browse the library</Button>
				<Button href="/community" variant="outline" class="rounded-full border-stone-200/60 px-5 text-[9px] tracking-wide">See the community</Button>
			</div>
		</div>
	</main>
</div>
