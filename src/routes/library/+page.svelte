<script lang="ts">
	import { fly } from 'svelte/transition';
	import { Badge } from '$lib/components/ui/badge/index.js';
	import { Button } from '$lib/components/ui/button/index.js';
	import { Input } from '$lib/components/ui/input/index.js';
	import {
		Select,
		SelectContent,
		SelectItem,
		SelectTrigger,
	} from '$lib/components/ui/select/index.js';
	import { Card, CardContent } from '$lib/components/ui/card/index.js';

	const products = [
		{ brand: 'bela', product: 'piri-piri', img: '/bela_lightlysmokedsardinesinevoowithpiripiri.png', name: 'Sardines, Piri-Piri', type: 'sardines', rating: 4.7, reviews: 42 },
		{ brand: 'fishwife', product: 'preserved-lemon', img: '/Screenshot_2025-03-19_at_3.22.26_PM.webp', name: 'Sardines, Preserved Lemon', type: 'sardines', rating: 4.4, reviews: 28 },
		{ brand: 'nuri', product: 'spiced', img: '/bela_lightlysmokedsardinesinevoowithpiripiri.png', name: 'Sardines, Spiced', type: 'sardines', rating: 4.5, reviews: 55 },
		{ brand: 'jose-gourmet', product: 'mackerel-olive-oil', img: '/Screenshot_2025-03-19_at_3.22.26_PM.webp', name: 'Mackerel, Olive Oil', type: 'mackerel', rating: 4.6, reviews: 31 },
		{ brand: 'matiz', product: 'galician-style', img: '/bela_lightlysmokedsardinesinevoowithpiripiri.png', name: 'Sardines, Galician Style', type: 'sardines', rating: 4.2, reviews: 19 },
		{ brand: 'scout', product: 'lemon-herb', img: '/Screenshot_2025-03-19_at_3.22.26_PM.webp', name: 'Tuna, Lemon & Herb', type: 'tuna', rating: 4.3, reviews: 37 },
		{ brand: 'cole', product: 'smoked', img: '/bela_lightlysmokedsardinesinevoowithpiripiri.png', name: 'Salmon, Smoked', type: 'salmon', rating: 4.1, reviews: 14 },
		{ brand: 'la-narval', product: 'anchovy-fillets', img: '/Screenshot_2025-03-19_at_3.22.26_PM.webp', name: 'Anchovies, in Oil', type: 'anchovies', rating: 4.8, reviews: 63 },
	];

	const types = ['all', 'sardines', 'mackerel', 'tuna', 'salmon', 'anchovies'];

	let activeType = $state('all');
	let query = $state('');
	let sort = $state('rating');

	const filtered = $derived(
		products
			.filter((p) => {
				const matchType = activeType === 'all' || p.type === activeType;
				const q = query.toLowerCase();
				const matchQuery = !q || p.brand.includes(q) || p.name.toLowerCase().includes(q) || p.type.includes(q);
				return matchType && matchQuery;
			})
			.sort((a, b) => b[sort as 'rating' | 'reviews'] - a[sort as 'rating' | 'reviews'])
	);
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link href="https://fonts.googleapis.com/css2?family=Syne:wght@800&display=swap" rel="stylesheet" />
	<title>Library · deens</title>
</svelte:head>

<div class="min-h-screen bg-stone-50">
	<!-- Navbar -->
	<nav class="sticky top-4 z-20 mx-auto flex w-4/5 items-center justify-between rounded-full border border-white/50 bg-white/70 px-8 py-2.5 shadow-sm backdrop-blur-xl">
		<a href="/" style="font-family: 'Syne', sans-serif;" class="text-sm font-extrabold uppercase tracking-tight text-stone-900">deens</a>
		<ul class="flex list-none items-center gap-8">
			<li><a href="/" class="text-[10px] uppercase tracking-widest text-stone-400 transition-colors hover:text-stone-900">Home</a></li>
			<li><a href="/library" class="text-[10px] uppercase tracking-widest text-stone-900 transition-colors hover:text-stone-400">Library</a></li>
			<li><a href="/community" class="text-[10px] uppercase tracking-widest text-stone-400 transition-colors hover:text-stone-900">Community</a></li>
		</ul>
		<Button size="sm" class="rounded-full px-4 text-[9px] tracking-wide">login</Button>
	</nav>

	<main class="mx-auto max-w-5xl px-6 pb-24 pt-12">
		<!-- Header -->
		<div in:fly={{ y: 12, duration: 400 }} class="mb-10 flex items-end justify-between">
			<div>
				<p class="mb-1.5 text-[9px] uppercase tracking-[0.35em] text-stone-400">Browse</p>
				<h1 style="font-family: 'Syne', sans-serif;" class="text-4xl font-extrabold uppercase tracking-tight text-stone-900">Library</h1>
			</div>
			<p class="text-[9px] uppercase tracking-widest text-stone-400">{filtered.length} tins</p>
		</div>

		<!-- Controls -->
		<div in:fly={{ y: 12, duration: 400, delay: 60 }} class="mb-8 flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
			<!-- Type filter chips -->
			<div class="flex flex-wrap gap-2">
				{#each types as t}
					<Button
						size="sm"
						variant={activeType === t ? 'default' : 'outline'}
						onclick={() => (activeType = t)}
						class="rounded-full border-stone-200/60 px-3.5 text-[9px] uppercase tracking-widest"
					>
						{t}
					</Button>
				{/each}
			</div>

			<!-- Search + sort -->
			<div class="flex items-center gap-3">
				<Input
					type="search"
					placeholder="Search…"
					bind:value={query}
					class="h-8 w-40 rounded-full border-stone-200/60 px-4 text-[10px]"
				/>
				<Select bind:value={sort}>
					<SelectTrigger class="h-8 w-36 rounded-full border-stone-200/60 px-4 text-[9px] uppercase tracking-widest">
						{sort === 'rating' ? 'Top rated' : 'Most reviewed'}
					</SelectTrigger>
					<SelectContent>
						<SelectItem value="rating" class="text-[10px]">Top rated</SelectItem>
						<SelectItem value="reviews" class="text-[10px]">Most reviewed</SelectItem>
					</SelectContent>
				</Select>
			</div>
		</div>

		<!-- Grid -->
		{#if filtered.length === 0}
			<div in:fly={{ y: 8, duration: 300 }} class="flex flex-col items-center gap-3 py-24 text-stone-400">
				<svg viewBox="0 0 24 24" width="28" height="28" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
					<circle cx="11" cy="11" r="8"/><path d="m21 21-4.35-4.35"/>
				</svg>
				<p class="text-[9px] uppercase tracking-widest">No tins found</p>
			</div>
		{:else}
			<div class="grid grid-cols-2 gap-4 sm:grid-cols-3 lg:grid-cols-4">
				{#each filtered as p, i (p.brand + p.product)}
					<a
						in:fly={{ y: 16, duration: 400, delay: i * 40 }}
						href="/library/{p.brand}/{p.product}"
						class="group"
					>
						<Card class="h-full ring-stone-900/5 transition-shadow hover:shadow-md">
							<CardContent class="flex flex-col p-4">
								<!-- Image -->
								<div class="mb-4 flex aspect-square items-center justify-center overflow-hidden rounded-xl bg-stone-50 p-4">
									<img
										src={p.img}
										alt="{p.brand} {p.name}"
										class="h-full w-full object-contain transition-transform duration-300 group-hover:scale-105"
										draggable="false"
									/>
								</div>

								<!-- Meta -->
								<p class="mb-0.5 text-[8px] uppercase tracking-[0.25em] text-stone-400">{p.brand}</p>
								<p class="mb-3 text-[11px] font-semibold leading-snug text-stone-800">{p.name}</p>

								<!-- Footer -->
								<div class="mt-auto flex items-center justify-between">
									<span class="text-[10px] tracking-tight text-amber-400">
										{'★'.repeat(Math.floor(p.rating))}{'☆'.repeat(5 - Math.floor(p.rating))}
									</span>
									<Badge variant="secondary" class="text-[8px] uppercase tracking-widest">
										{p.type}
									</Badge>
								</div>
							</CardContent>
						</Card>
					</a>
				{/each}
			</div>
		{/if}
	</main>
</div>
