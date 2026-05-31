<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import { goto } from '$app/navigation';
	import { fade } from 'svelte/transition';

	const COLS = 20;
	const ROWS = 100;
	const CELL = 160;
	const GAP = 48;
	const SRCS = [
		{ src: '/bela_lightlysmokedsardinesinevoowithpiripiri.png', brand: 'bela', desc: 'sardines (piri-piri)', product: 'piri-piri' },
		{ src: '/Screenshot_2025-03-19_at_3.22.26_PM.webp', brand: 'fishwife', desc: 'sardines (preserved lemon)', product: 'preserved-lemon' }
	];
	const CANVAS_W = COLS * CELL + (COLS + 1) * GAP;
	const CANVAS_H = ROWS * CELL + (ROWS + 1) * GAP;
	const FRICTION = 0.92;
	const CURVE_FRICTION = 0.004;
	const MAX_SCALE = 4;

	const tiles = Array.from({ length: COLS * ROWS }, () => {
		const item = SRCS[Math.floor(Math.random() * SRCS.length)];
		return { ...item };
	});

	let viewportEl: HTMLDivElement;
	let canvasEl: HTMLDivElement;
	let tx = 0;
	let ty = 0;
	let scale = $state(1);
	let initTx = 0;
	let initTy = 0;
	let showWelcome = $state(true);
	let isDragging = $state(false);
	let pointerMoved = false;
	let pendingNav: string | null = null;
	let startX = 0;
	let startY = 0;
	let lastX = 0;
	let lastY = 0;
	let velX = 0;
	let velY = 0;
	let rafId = 0;

	const samples: { x: number; y: number; t: number }[] = [];

	function clamp(v: number, lo: number, hi: number) {
		return v < lo ? lo : v > hi ? hi : v;
	}

	function getBounds() {
		const vw = viewportEl.clientWidth;
		const vh = viewportEl.clientHeight;
		const sw = CANVAS_W * scale;
		const sh = CANVAS_H * scale;
		return {
			x0: Math.min(0, -(sw - vw)),
			y0: Math.min(0, -(sh - vh)),
			x1: Math.max(0, vw - sw),
			y1: Math.max(0, vh - sh)
		};
	}

	function applyTransform() {
		canvasEl.style.transform = `translate(${Math.round(tx)}px,${Math.round(ty)}px) scale(${scale})`;
	}

	function pan(dx: number, dy: number) {
		const { x0, y0, x1, y1 } = getBounds();
		tx = clamp(tx + dx, x0, x1);
		ty = clamp(ty + dy, y0, y1);
		applyTransform();
	}

	function tick() {
		const speed = Math.hypot(velX, velY);
		if (speed < 0.5) return;
		const factor = Math.max(0, FRICTION - CURVE_FRICTION * speed);
		velX *= factor;
		velY *= factor;
		pan(velX, velY);
		rafId = requestAnimationFrame(tick);
	}

	function onPointerDown(e: PointerEvent) {
		if (showWelcome) return;
		cancelAnimationFrame(rafId);
		isDragging = true;
		pointerMoved = false;
		startX = e.clientX;
		startY = e.clientY;
		lastX = e.clientX;
		lastY = e.clientY;
		const tileEl = (e.target as Element).closest('[data-href]');
		pendingNav = tileEl ? tileEl.getAttribute('data-href') : null;
		velX = velY = 0;
		samples.length = 0;
		samples.push({ x: e.clientX, y: e.clientY, t: Date.now() });
		viewportEl.setPointerCapture(e.pointerId);
	}

	function onPointerMove(e: PointerEvent) {
		if (!isDragging) return;
		if (!pointerMoved && Math.hypot(e.clientX - startX, e.clientY - startY) > 6) pointerMoved = true;
		const now = Date.now();
		samples.push({ x: e.clientX, y: e.clientY, t: now });
		while (samples.length > 1 && now - samples[0].t > 100) samples.shift();
		pan(e.clientX - lastX, e.clientY - lastY);
		lastX = e.clientX;
		lastY = e.clientY;
	}

	function onPointerUp() {
		if (!isDragging) return;
		isDragging = false;
		const now = Date.now();
		const cutoff = now - 100;
		const old = samples.find((s) => s.t >= cutoff) ?? samples[0];
		const last = samples.at(-1);
		if (old && last && last.t > old.t) {
			const dt = last.t - old.t;
			velX = ((last.x - old.x) / dt) * 16;
			velY = ((last.y - old.y) / dt) * 16;
		}
		samples.length = 0;
		if (!pointerMoved && pendingNav) goto(pendingNav);
		else rafId = requestAnimationFrame(tick);
		pendingNav = null;
	}

	function handleWheel(e: WheelEvent) {
		e.preventDefault();
		if (showWelcome) return;
		cancelAnimationFrame(rafId);
		const minScale = Math.max(
			viewportEl.clientWidth / CANVAS_W,
			viewportEl.clientHeight / CANVAS_H
		);
		const factor = e.deltaY < 0 ? 1.1 : 1 / 1.1;
		const newScale = clamp(scale * factor, minScale, MAX_SCALE);
		const ratio = newScale / scale;
		tx = e.clientX - (e.clientX - tx) * ratio;
		ty = e.clientY - (e.clientY - ty) * ratio;
		scale = newScale;
		const { x0, y0, x1, y1 } = getBounds();
		tx = clamp(tx, x0, x1);
		ty = clamp(ty, y0, y1);
		applyTransform();
	}

	function zoomStep(factor: number) {
		cancelAnimationFrame(rafId);
		const minScale = Math.max(viewportEl.clientWidth / CANVAS_W, viewportEl.clientHeight / CANVAS_H);
		const cx = viewportEl.clientWidth / 2;
		const cy = viewportEl.clientHeight / 2;
		const newScale = clamp(scale * factor, minScale, MAX_SCALE);
		const ratio = newScale / scale;
		tx = cx - (cx - tx) * ratio;
		ty = cy - (cy - ty) * ratio;
		scale = newScale;
		const { x0, y0, x1, y1 } = getBounds();
		tx = clamp(tx, x0, x1);
		ty = clamp(ty, y0, y1);
		applyTransform();
	}

	function resetView() {
		cancelAnimationFrame(rafId);
		tx = initTx;
		ty = initTy;
		scale = 1;
		applyTransform();
	}

	onMount(() => {
		tx = viewportEl.clientWidth / 2 - CANVAS_W / 2;
		ty = viewportEl.clientHeight * 0.42 - CANVAS_H / 2;
		initTx = tx;
		initTy = ty;
		applyTransform();
		viewportEl.addEventListener('wheel', handleWheel, { passive: false });
		return () => viewportEl.removeEventListener('wheel', handleWheel);
	});

	onDestroy(() => {
		if (browser) cancelAnimationFrame(rafId);
	});
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link
		href="https://fonts.googleapis.com/css2?family=Syne:wght@800&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div class="relative h-screen overflow-hidden bg-stone-50">
	<!-- Navbar: light mode -->
	{#if !showWelcome}
		<nav class="absolute left-1/2 top-4 z-20 flex w-4/5 -translate-x-1/2 items-center justify-between rounded-full border border-white/50 bg-white/40 px-8 py-2.5 shadow-sm backdrop-blur-xl">
			<a href="/" style="font-family: 'Syne', sans-serif;" class="text-sm font-extrabold uppercase tracking-tight text-stone-900">deens</a>
			<ul class="flex list-none items-center gap-8">
				<li><a href="/" class="text-[10px] uppercase tracking-widest text-stone-900 transition-colors hover:text-stone-400">Home</a></li>
				<li><a href="/library" class="text-[10px] uppercase tracking-widest text-stone-400 transition-colors hover:text-stone-900">Library</a></li>
				<li><a href="/community" class="text-[10px] uppercase tracking-widest text-stone-400 transition-colors hover:text-stone-900">Community</a></li>
			</ul>
			<button class="inline-flex items-center rounded-full bg-blue-600 px-4 py-1.5 text-[9px] tracking-wide text-white transition-colors hover:bg-blue-700">login</button>
		</nav>
	{/if}

	<!-- Navbar: dark/welcome mode -->
	{#if showWelcome}
		<nav class="absolute left-1/2 top-4 z-20 flex w-4/5 -translate-x-1/2 items-center justify-between rounded-full border border-white/10 bg-white/5 px-8 py-2.5 backdrop-blur-xl">
			<a href="/" style="font-family: 'Syne', sans-serif;" class="text-sm font-extrabold uppercase tracking-tight text-white">deens</a>
			<ul class="flex list-none items-center gap-8">
				<li><a href="/" class="text-[10px] uppercase tracking-widest text-white transition-colors hover:text-white/60">Home</a></li>
				<li><a href="/library" class="text-[10px] uppercase tracking-widest text-white/50 transition-colors hover:text-white">Library</a></li>
				<li><a href="/community" class="text-[10px] uppercase tracking-widest text-white/50 transition-colors hover:text-white">Community</a></li>
			</ul>
			<button class="inline-flex items-center rounded-full border border-white/20 bg-white/10 px-4 py-1.5 text-[9px] tracking-wide text-white transition-colors hover:bg-white/20">login</button>
		</nav>
	{/if}

	<!-- Canvas viewport -->
	<div
		bind:this={viewportEl}
		class="h-full select-none overflow-hidden"
		class:cursor-grabbing={isDragging}
		class:cursor-grab={!isDragging}
		onpointerdown={onPointerDown}
		onpointermove={onPointerMove}
		onpointerup={onPointerUp}
		onpointerleave={onPointerUp}
	>
		<div
			bind:this={canvasEl}
			style="
        width: {CANVAS_W}px;
        height: {CANVAS_H}px;
        transform-origin: 0 0;
        display: grid;
        grid-template-columns: repeat({COLS}, {CELL}px);
        gap: {GAP}px;
        padding: {GAP}px;
      "
		>
			{#each tiles as tile}
				<div
					role="button"
					tabindex="0"
					data-href="/library/{tile.brand}/{tile.product}"
					onkeydown={(e) => e.key === 'Enter' && goto(`/library/${tile.brand}/${tile.product}`)}
					style="
            width: {CELL}px;
            content-visibility: auto;
            contain-intrinsic-size: {CELL}px {CELL + 36}px;
            cursor: pointer;
          "
					class="group"
				>
					<img
						src={tile.src}
						alt=""
						width={CELL}
						height={CELL}
						draggable="false"
						style="width:{CELL}px;height:{CELL}px;display:block;object-fit:contain;"
						class="contrast-100 transition-[filter] duration-200 group-hover:contrast-[1.2]"
					/>
					<p
						title="{tile.brand}, {tile.desc}"
						style="margin-top:8px;font-size:9px;color:#a8a29e;text-align:center;line-height:1.3;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;width:{CELL}px;"
					>
						{tile.brand}, {tile.desc}
					</p>
				</div>
			{/each}
		</div>
	</div>

	<!-- Zoom controls -->
	{#if !showWelcome}
		<div class="absolute bottom-6 right-6 z-20 flex flex-row items-center rounded-xl border border-stone-200 bg-white/80 px-1.5 shadow-sm backdrop-blur-sm">
			<button
				onclick={() => zoomStep(0.8)}
				class="flex h-8 w-8 items-center justify-center text-stone-500 transition-colors hover:text-stone-900"
				aria-label="Zoom out"
			>
				<svg viewBox="0 0 16 16" width="13" height="13" class="fill-current"><path d="M2 8a.75.75 0 0 1 .75-.75h10.5a.75.75 0 0 1 0 1.5H2.75A.75.75 0 0 1 2 8Z"/></svg>
			</button>
			<span class="select-none px-1 text-[9px] tabular-nums text-stone-400">{Math.round(scale * 100)}%</span>
			<button
				onclick={() => zoomStep(1.25)}
				class="flex h-8 w-8 items-center justify-center text-stone-500 transition-colors hover:text-stone-900"
				aria-label="Zoom in"
			>
				<svg viewBox="0 0 16 16" width="13" height="13" class="fill-current"><path d="M8 2a.75.75 0 0 1 .75.75v4.5h4.5a.75.75 0 0 1 0 1.5h-4.5v4.5a.75.75 0 0 1-1.5 0v-4.5h-4.5a.75.75 0 0 1 0-1.5h4.5v-4.5A.75.75 0 0 1 8 2Z"/></svg>
			</button>
			<div class="mx-1 h-5 w-px bg-stone-200"></div>
			<button
				onclick={resetView}
				class="flex h-8 w-8 items-center justify-center text-stone-500 transition-colors hover:text-stone-900"
				aria-label="Reset view"
			>
				<svg viewBox="0 0 16 16" width="13" height="13" class="fill-current"><path d="M8 1a7 7 0 1 0 0 14A7 7 0 0 0 8 1ZM0 8a8 8 0 1 1 16 0A8 8 0 0 1 0 8Zm8-3.25a.75.75 0 0 1 .75.75v2.69l1.28 1.28a.75.75 0 1 1-1.06 1.06l-1.5-1.5a.75.75 0 0 1-.22-.53V5.5A.75.75 0 0 1 8 4.75Z"/></svg>
			</button>
		</div>
	{/if}

	<!-- Welcome overlay -->
	{#if showWelcome}
		<div
			out:fade={{ duration: 400 }}
			class="absolute inset-0 z-10 flex flex-col items-center justify-center bg-black/90 backdrop-blur-[2px]"
			onclick={() => (showWelcome = false)}
			role="button"
			tabindex="0"
			onkeydown={(e) => e.key === 'Escape' && (showWelcome = false)}
		>
			<h1
				style="font-family: 'Syne', sans-serif; animation: blur-fade 0.8s ease-out both;"
				class="text-6xl font-extrabold uppercase tracking-[0.1em] text-white"
			>
				deens
			</h1>
			<p
				style="animation: blur-fade 0.8s ease-out both; animation-delay: 0.1s;"
				class="mt-2 text-[9px] uppercase tracking-[0.35em] text-white/65"
			>
				Library of tinned fish
			</p>
			<div style="animation: blur-fade 0.8s ease-out both; animation-delay: 0.22s;" class="mt-9 flex items-center gap-3">
				<a
					href="/about"
					onclick={(e) => e.stopPropagation()}
					class="inline-flex items-center rounded-full border border-white/30 bg-white/15 px-6 py-2.5 text-[9px] tracking-wide text-white backdrop-blur-sm transition-colors hover:bg-white/25"
				>
					about
				</a>
				<button
					onclick={() => (showWelcome = false)}
					class="inline-flex items-center rounded-full border border-white/30 bg-white/15 px-6 py-2.5 text-[9px] tracking-wide text-white backdrop-blur-sm transition-colors hover:bg-white/25"
				>
					view library
				</button>
			</div>
			<p
				style="animation: blur-fade 0.8s ease-out both; animation-delay: 0.34s;"
				class="mt-6 flex items-center justify-center gap-1.5 text-[9px] text-white/40"
			>
				made by
				<a
					href="https://github.com/lhris"
					target="_blank"
					rel="noopener noreferrer"
					onclick={(e) => e.stopPropagation()}
					class="flex items-center gap-1 text-white/60 transition-colors hover:text-white"
				>
					<svg viewBox="0 0 16 16" width="12" height="12" class="fill-current">
						<path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/>
					</svg>
					lhris
				</a>
			</p>
		</div>
	{/if}
</div>

<style>
	@keyframes blur-fade {
		from {
			filter: blur(14px);
			opacity: 0;
		}
		to {
			filter: blur(0px);
			opacity: 1;
		}
	}
</style>
