<script lang="ts">
	import { fly } from 'svelte/transition';
	import { Badge } from '$lib/components/ui/badge/index.js';
	import { Button } from '$lib/components/ui/button/index.js';
	import { Card, CardContent } from '$lib/components/ui/card/index.js';

	const feed = [
		{ initials: 'LA', user: 'la-narval', date: 'Today', rating: 5, brand: 'la-narval', product: 'anchovy-fillets', tin: 'Anchovies, in Oil', note: 'Silky and intensely savoury. Dissolved into pasta butter sauce and it was perfect.' },
		{ initials: 'JK', user: 'jkuo', date: 'Today', rating: 4, brand: 'jose-gourmet', product: 'mackerel-olive-oil', tin: 'Mackerel, Olive Oil', note: 'Meaty and rich. A bit oily but that\'s half the point. Great on sourdough.' },
		{ initials: 'MR', user: 'mrose', date: 'Yesterday', rating: 5, brand: 'nuri', product: 'spiced', tin: 'Sardines, Spiced', note: 'Best sardine I\'ve had in a long time. Complex spice, clean finish.' },
		{ initials: 'CL', user: 'lhris', date: 'Yesterday', rating: 5, brand: 'bela', product: 'piri-piri', tin: 'Sardines, Piri-Piri', note: 'Perfect on ritz with a squeeze of lemon. The piri-piri heat is subtle but lingers nicely.' },
		{ initials: 'SP', user: 'spen', date: 'May 28', rating: 3, brand: 'scout', product: 'lemon-herb', tin: 'Tuna, Lemon & Herb', note: 'Fine but nothing special. Lemon flavour reads a bit artificial to me.' },
		{ initials: 'MR', user: 'mrose', date: 'Apr 28', rating: 4, brand: 'fishwife', product: 'preserved-lemon', tin: 'Sardines, Preserved Lemon', note: 'Really good. Oil quality is excellent. Wish there was a touch more heat.' },
		{ initials: 'JK', user: 'jkuo', date: 'Mar 15', rating: 5, brand: 'bela', product: 'piri-piri', tin: 'Sardines, Piri-Piri', note: 'Top tier. These are my go-to now. Great on toast with butter.' },
	];

	const contributors = [
		{ initials: 'MR', user: 'mrose', tins: 24, avg: 4.3 },
		{ initials: 'JK', user: 'jkuo', tins: 19, avg: 4.6 },
		{ initials: 'CL', user: 'lhris', tins: 17, avg: 4.8 },
		{ initials: 'SP', user: 'spen', tins: 11, avg: 3.9 },
		{ initials: 'LA', user: 'la-narval', tins: 8, avg: 4.5 },
	];
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link href="https://fonts.googleapis.com/css2?family=Syne:wght@800&display=swap" rel="stylesheet" />
	<title>Community · deens</title>
</svelte:head>

<div class="min-h-screen bg-stone-50">
	<!-- Navbar -->
	<nav class="sticky top-4 z-20 mx-auto flex w-4/5 items-center justify-between rounded-full border border-white/50 bg-white/70 px-8 py-2.5 shadow-sm backdrop-blur-xl">
		<a href="/" style="font-family: 'Syne', sans-serif;" class="text-sm font-extrabold uppercase tracking-tight text-stone-900">deens</a>
		<ul class="flex list-none items-center gap-8">
			<li><a href="/" class="text-[10px] uppercase tracking-widest text-stone-400 transition-colors hover:text-stone-900">Home</a></li>
			<li><a href="/library" class="text-[10px] uppercase tracking-widest text-stone-400 transition-colors hover:text-stone-900">Library</a></li>
			<li><a href="/community" class="text-[10px] uppercase tracking-widest text-stone-900 transition-colors hover:text-stone-400">Community</a></li>
		</ul>
		<Button size="sm" class="rounded-full px-4 text-[9px] tracking-wide">login</Button>
	</nav>

	<main class="mx-auto max-w-5xl px-6 pb-24 pt-12">
		<!-- Header -->
		<div in:fly={{ y: 12, duration: 400 }} class="mb-10">
			<p class="mb-1.5 text-[9px] uppercase tracking-[0.35em] text-stone-400">What people are eating</p>
			<h1 style="font-family: 'Syne', sans-serif;" class="text-4xl font-extrabold uppercase tracking-tight text-stone-900">Community</h1>
		</div>

		<div class="grid grid-cols-1 gap-10 lg:grid-cols-[1fr_260px]">
			<!-- Feed -->
			<div in:fly={{ y: 16, duration: 400, delay: 60 }} class="space-y-3">
				<p class="mb-5 text-[9px] uppercase tracking-widest text-stone-400">Recent activity</p>
				{#each feed as entry, i}
					<Card class="ring-stone-900/5">
						<CardContent class="flex gap-5 p-5">
							<!-- Avatar -->
							<div class="flex h-9 w-9 shrink-0 items-center justify-center rounded-full bg-stone-100 text-[10px] font-bold text-stone-600">
								{entry.initials}
							</div>
							<div class="flex-1 min-w-0">
								<div class="flex flex-wrap items-center gap-2 mb-2">
									<span class="text-xs font-semibold text-stone-800">{entry.user}</span>
									<span class="text-[9px] text-stone-400">tried</span>
									<a
										href="/library/{entry.brand}/{entry.product}"
										class="text-[9px] font-medium text-stone-700 underline-offset-2 hover:underline"
									>
										{entry.tin}
									</a>
									<span class="ml-auto text-[10px] tracking-tight text-amber-400 shrink-0">
										{'★'.repeat(entry.rating)}{'☆'.repeat(5 - entry.rating)}
									</span>
								</div>
								<p class="text-sm leading-6 text-stone-500">{entry.note}</p>
								<p class="mt-2 text-[9px] text-stone-300">{entry.date}</p>
							</div>
						</CardContent>
					</Card>
				{/each}
			</div>

			<!-- Sidebar -->
			<div in:fly={{ y: 16, duration: 400, delay: 120 }} class="space-y-8">
				<!-- Top contributors -->
				<div>
					<p class="mb-4 text-[9px] uppercase tracking-widest text-stone-400">Top tasters</p>
					<div class="space-y-2">
						{#each contributors as c, i}
							<Card class="ring-stone-900/5">
								<CardContent class="flex items-center gap-3 px-4 py-3">
									<span class="w-4 text-[9px] tabular-nums text-stone-300">{i + 1}</span>
									<div class="flex h-7 w-7 shrink-0 items-center justify-center rounded-full bg-stone-100 text-[9px] font-bold text-stone-600">
										{c.initials}
									</div>
									<span class="flex-1 text-[11px] font-medium text-stone-700">{c.user}</span>
									<Badge variant="secondary" class="text-[8px] tabular-nums">{c.tins} tins</Badge>
								</CardContent>
							</Card>
						{/each}
					</div>
				</div>

				<!-- CTA -->
				<Card class="ring-stone-900/5 bg-stone-50">
					<CardContent class="flex flex-col items-start gap-3 p-5">
						<p class="text-[9px] uppercase tracking-widest text-stone-400">Join the list</p>
						<p class="text-sm leading-6 text-stone-600">Log tins, rate them, and see what the community is eating.</p>
						<Button size="sm" class="rounded-full px-4 text-[9px] tracking-wide">Create account</Button>
					</CardContent>
				</Card>
			</div>
		</div>
	</main>
</div>
