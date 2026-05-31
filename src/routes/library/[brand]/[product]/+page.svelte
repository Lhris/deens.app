<script lang="ts">
	import { page } from '$app/state';
	import { fly } from 'svelte/transition';

	const brand = $derived(page.params.brand);
	const product = $derived(page.params.product);
	const productLabel = $derived(product.split('-').map((w: string) => w[0].toUpperCase() + w.slice(1)).join(' '));

	const nutrition = [
		{ label: 'Calories', value: '190', unit: 'kcal' },
		{ label: 'Protein', value: '22', unit: 'g' },
		{ label: 'Total Fat', value: '11', unit: 'g' },
		{ label: 'Sat. Fat', value: '2.5', unit: 'g' },
		{ label: 'Sodium', value: '280', unit: 'mg' },
		{ label: 'Omega-3', value: '1.8', unit: 'g' },
	];

	const ingredients = [
		'Sardines (Sardina pilchardus)',
		'Extra Virgin Olive Oil',
		'Piri-Piri Pepper (Capsicum frutescens)',
		'Salt',
	];

	const log = [
		{ initials: 'CL', user: 'lhris', date: 'May 12, 2025', rating: 5, note: 'Perfect on ritz with a squeeze of lemon. The piri-piri heat is subtle but lingers nicely.' },
		{ initials: 'MR', user: 'mrose', date: 'Apr 28, 2025', rating: 4, note: 'Really good. Oil quality is excellent. Wish there was a touch more heat.' },
		{ initials: 'JK', user: 'jkuo', date: 'Mar 15, 2025', rating: 5, note: 'Top tier. These are my go-to now. Great on toast with butter.' },
	];
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link href="https://fonts.googleapis.com/css2?family=Syne:wght@800&display=swap" rel="stylesheet" />
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
		<button class="inline-flex items-center rounded-full bg-blue-600 px-4 py-1.5 text-[9px] tracking-wide text-white transition-colors hover:bg-blue-700">login</button>
	</nav>

	<main class="mx-auto max-w-5xl px-6 pb-24 pt-12">
		<!-- Breadcrumb -->
		<p in:fly={{ y: 12, duration: 400, delay: 0 }} class="mb-8 text-[9px] uppercase tracking-widest text-stone-400">
			<a href="/" class="transition-colors hover:text-stone-700">Home</a>
			<span class="mx-2">·</span>
			<a href="/library" class="transition-colors hover:text-stone-700">Library</a>
			<span class="mx-2">·</span>
			<span class="text-stone-600">{brand}</span>
		</p>

		<!-- Top section: photo + nutrition -->
		<div class="grid grid-cols-2 gap-16">
			<!-- Photo -->
			<div in:fly={{ y: 16, duration: 500, delay: 80 }} class="flex items-center justify-center rounded-2xl border border-stone-100 bg-stone-50 p-10">
				<img
					src="/bela_lightlysmokedsardinesinevoowithpiripiri.png"
					alt="{brand} {productLabel}"
					class="h-72 w-72 object-contain"
				/>
			</div>

			<!-- Nutrition -->
			<div in:fly={{ y: 16, duration: 500, delay: 160 }} class="flex flex-col justify-center">
				<p class="mb-1 text-[9px] uppercase tracking-[0.3em] text-stone-400">{brand}</p>
				<h1 style="font-family: 'Syne', sans-serif;" class="mb-6 text-3xl font-extrabold uppercase tracking-tight text-stone-900">
					{productLabel}
				</h1>
				<p class="mb-4 text-[9px] uppercase tracking-widest text-stone-400">Nutrition · per 85g serving</p>
				<div class="grid grid-cols-3 gap-3">
					{#each nutrition as n}
						<div class="rounded-xl border border-stone-100 bg-stone-50 p-4 text-center">
							<p class="text-xl font-bold text-stone-900">{n.value}<span class="text-xs font-normal text-stone-400">{n.unit}</span></p>
							<p class="mt-1 text-[8px] uppercase tracking-widest text-stone-400">{n.label}</p>
						</div>
					{/each}
				</div>
			</div>
		</div>

		<!-- Middle section: description + ingredients -->
		<div class="mt-16 grid grid-cols-2 gap-16">
			<!-- Description -->
			<div in:fly={{ y: 16, duration: 500, delay: 240 }}>
				<p class="mb-4 text-[9px] uppercase tracking-widest text-stone-400">About</p>
				<p class="text-sm leading-7 text-stone-600">
					Bela's lightly smoked sardines in extra virgin olive oil with piri-piri bring a gentle heat to one of Portugal's finest tinned fish traditions. Sourced from the Atlantic coast and hand-packed in Póvoa de Varzim, each tin holds whole fish with firm flesh and clean, ocean-forward flavour.
				</p>
				<p class="mt-4 text-sm leading-7 text-stone-600">
					The piri-piri adds warmth without overwhelming the natural richness of the sardine. Best enjoyed at room temperature on good bread, with a squeeze of lemon.
				</p>
			</div>

			<!-- Ingredients -->
			<div in:fly={{ y: 16, duration: 500, delay: 300 }}>
				<p class="mb-4 text-[9px] uppercase tracking-widest text-stone-400">Ingredients</p>
				<ol class="space-y-3">
					{#each ingredients as ing, i}
						<li class="flex items-start gap-4">
							<span class="mt-0.5 flex h-5 w-5 shrink-0 items-center justify-center rounded-full bg-stone-100 text-[8px] font-bold text-stone-500">{i + 1}</span>
							<span class="text-sm text-stone-600">{ing}</span>
						</li>
					{/each}
				</ol>
			</div>
		</div>

		<!-- Activity log -->
		<div in:fly={{ y: 16, duration: 500, delay: 380 }} class="mt-20">
			<p class="mb-6 text-[9px] uppercase tracking-widest text-stone-400">Tried by</p>
			<div class="space-y-4">
				{#each log as entry}
					<div class="flex gap-5 rounded-2xl border border-stone-100 bg-stone-50 p-6">
						<!-- Avatar -->
						<div class="flex h-9 w-9 shrink-0 items-center justify-center rounded-full bg-stone-200 text-[10px] font-bold text-stone-600">
							{entry.initials}
						</div>
						<div class="flex-1">
							<div class="flex items-center gap-3">
								<span class="text-xs font-semibold text-stone-800">{entry.user}</span>
								<span class="text-[9px] text-stone-400">{entry.date}</span>
								<span class="ml-auto text-xs tracking-tight text-amber-400">
									{'★'.repeat(entry.rating)}{'☆'.repeat(5 - entry.rating)}
								</span>
							</div>
							<p class="mt-1.5 text-sm leading-6 text-stone-500">{entry.note}</p>
						</div>
					</div>
				{/each}
			</div>
		</div>
	</main>
</div>
