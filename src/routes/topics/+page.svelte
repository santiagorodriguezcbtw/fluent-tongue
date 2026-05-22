<svelte:options runes={true} />

<script lang="ts">
	import { browser } from '$app/environment';
	import { onMount } from 'svelte';

	type Level = 'Beginner' | 'Intermediate' | 'Advanced';
	type Topic = {
		id: number;
		title: string;
		description: string;
		level: Level;
		lessons: number;
		progress: number;
		favorite: boolean;
		tags: string[];
	};

	const STORAGE_KEY = 'topics-page';
	const seed: Topic[] = [
		{
			id: 1,
			title: 'Greetings',
			description: 'Common introductions, polite responses, and daily phrases.',
			level: 'Beginner',
			lessons: 8,
			progress: 65,
			favorite: true,
			tags: ['hello', 'intro', 'polite']
		},
		{
			id: 2,
			title: 'Travel',
			description: 'Useful vocabulary for airports, hotels, and directions.',
			level: 'Intermediate',
			lessons: 12,
			progress: 35,
			favorite: false,
			tags: ['hotel', 'airport', 'maps']
		},
		{
			id: 3,
			title: 'Work',
			description: 'Meetings, tasks, schedules, and office communication.',
			level: 'Advanced',
			lessons: 14,
			progress: 20,
			favorite: false,
			tags: ['office', 'emails', 'meetings']
		},
		{
			id: 4,
			title: 'Food',
			description: 'Ordering meals, ingredients, and restaurant conversations.',
			level: 'Beginner',
			lessons: 10,
			progress: 80,
			favorite: true,
			tags: ['menu', 'restaurant', 'drinks']
		}
	];

	let topics = $state<Topic[]>(seed);
	let search = $state('');
	let level = $state<'All' | Level>('All');
	let favoritesOnly = $state(false);
	let selectedId = $state(1);

	onMount(() => {
		if (!browser) return;

		const saved = localStorage.getItem(STORAGE_KEY);

		if (!saved) return;

		try {
			const parsed = JSON.parse(saved) as Topic[];
			if (Array.isArray(parsed) && parsed.length) {
				topics = parsed;
				selectedId = parsed[0].id;
			}
		} catch {
			localStorage.removeItem(STORAGE_KEY);
		}
	});

	const filteredTopics = $derived(
		topics.filter((topic) => {
			const q = search.toLowerCase();
			const matchesSearch =
				topic.title.toLowerCase().includes(q) ||
				topic.description.toLowerCase().includes(q) ||
				topic.tags.some((tag) => tag.toLowerCase().includes(q));
			const matchesLevel = level === 'All' || topic.level === level;
			const matchesFavorite = !favoritesOnly || topic.favorite;

			return matchesSearch && matchesLevel && matchesFavorite;
		})
	);

	const selectedTopic = $derived(
		filteredTopics.find((topic) => topic.id === selectedId) ?? filteredTopics[0] ?? null
	);

	const totalLessons = $derived(topics.reduce((total, topic) => total + topic.lessons, 0));
	const favoriteCount = $derived(topics.filter((topic) => topic.favorite).length);
	const averageProgress = $derived(
		topics.length
			? Math.round(topics.reduce((total, topic) => total + topic.progress, 0) / topics.length)
			: 0
	);

	$effect(() => {
		if (selectedTopic && selectedTopic.id !== selectedId) {
			selectedId = selectedTopic.id;
		}
	});

	$effect(() => {
		if (!browser) return;
		localStorage.setItem(STORAGE_KEY, JSON.stringify(topics));
	});

	function updateTopic(id: number, updater: (topic: Topic) => Topic) {
		topics = topics.map((topic) => (topic.id === id ? updater(topic) : topic));
	}

	function toggleFavorite(id: number) {
		updateTopic(id, (topic) => ({ ...topic, favorite: !topic.favorite }));
	}

	function practice(id: number) {
		updateTopic(id, (topic) => ({
			...topic,
			progress: Math.min(100, topic.progress + 10)
		}));
	}

	function reset(id: number) {
		updateTopic(id, (topic) => ({ ...topic, progress: 0 }));
	}

	function selectTopic(id: number) {
		selectedId = id;
	}

	function badge(level: Level) {
		if (level === 'Beginner')
			return 'bg-emerald-500/15 text-emerald-300 ring-1 ring-emerald-500/25';
		if (level === 'Intermediate') return 'bg-sky-500/15 text-sky-300 ring-1 ring-sky-500/25';
		return 'bg-fuchsia-500/15 text-fuchsia-300 ring-1 ring-fuchsia-500/25';
	}

	function progressTone(progress: number) {
		if (progress >= 80) return 'bg-emerald-400';
		if (progress >= 40) return 'bg-cyan-400';
		return 'bg-amber-400';
	}
</script>

<svelte:head>
	<title>Topics</title>
</svelte:head>

<div class="relative min-h-screen overflow-hidden bg-slate-950 text-slate-100">
	<div class="absolute inset-0">
		<div class="absolute top-0 left-0 h-72 w-72 rounded-full bg-cyan-500/12 blur-3xl"></div>
		<div class="absolute top-24 right-0 h-80 w-80 rounded-full bg-fuchsia-500/10 blur-3xl"></div>
		<div class="absolute bottom-0 left-1/3 h-72 w-72 rounded-full bg-sky-500/10 blur-3xl"></div>
	</div>

	<div class="relative mx-auto max-w-7xl px-4 py-8 sm:px-6 lg:px-8">
		<div class="flex flex-col gap-6 lg:flex-row lg:items-end lg:justify-between">
			<div>
				<p class="text-sm font-medium tracking-[0.25em] text-cyan-300 uppercase">Learning Hub</p>
				<h1 class="mt-2 text-3xl font-bold tracking-tight sm:text-4xl">Topics</h1>
				<p class="mt-3 max-w-2xl text-sm text-slate-300 sm:text-base">
					Explora temas, continúa tus prácticas y sigue tu progreso.
				</p>
			</div>

			<div class="grid gap-3 sm:grid-cols-3">
				<div class="rounded-2xl border border-white/10 bg-white/5 px-4 py-3 backdrop-blur">
					<p class="text-xs text-slate-400">Promedio</p>
					<p class="mt-1 text-xl font-semibold">{averageProgress}%</p>
				</div>
				<div class="rounded-2xl border border-white/10 bg-white/5 px-4 py-3 backdrop-blur">
					<p class="text-xs text-slate-400">Favoritos</p>
					<p class="mt-1 text-xl font-semibold">{favoriteCount}</p>
				</div>
				<div class="rounded-2xl border border-white/10 bg-white/5 px-4 py-3 backdrop-blur">
					<p class="text-xs text-slate-400">Lecciones</p>
					<p class="mt-1 text-xl font-semibold">{totalLessons}</p>
				</div>
			</div>
		</div>

		<div class="mt-8 grid gap-6 lg:grid-cols-[1.15fr,0.85fr]">
			<section
				class="rounded-3xl border border-white/10 bg-slate-900/70 p-4 shadow-2xl shadow-black/30 backdrop-blur sm:p-6"
			>
				<div class="grid gap-3 lg:grid-cols-[1fr,180px,auto]">
					<input
						bind:value={search}
						type="text"
						placeholder="Buscar temas o tags..."
						class="rounded-2xl border border-white/10 bg-slate-950/80 px-4 py-3 text-sm text-white outline-none placeholder:text-slate-500 focus:border-cyan-400"
					/>

					<select
						bind:value={level}
						class="rounded-2xl border border-white/10 bg-slate-950/80 px-4 py-3 text-sm text-white outline-none focus:border-cyan-400"
					>
						<option value="All">Todos los niveles</option>
						<option value="Beginner">Beginner</option>
						<option value="Intermediate">Intermediate</option>
						<option value="Advanced">Advanced</option>
					</select>

					<label
						class="flex items-center gap-3 rounded-2xl border border-white/10 bg-slate-950/80 px-4 py-3 text-sm text-slate-200"
					>
						<input bind:checked={favoritesOnly} type="checkbox" class="h-4 w-4 accent-cyan-400" />
						Solo favoritos
					</label>
				</div>

				<div class="mt-6 grid gap-4 md:grid-cols-2">
					{#each filteredTopics as topic}
						<article
							class={`rounded-3xl border p-5 transition ${
								selectedId === topic.id
									? 'border-cyan-400/50 bg-slate-800/95 ring-1 ring-cyan-400/30'
									: 'border-white/10 bg-slate-950/70 hover:border-white/20 hover:bg-slate-900/90'
							}`}
						>
							<div class="flex items-start justify-between gap-3">
								<div>
									<h2 class="text-lg font-semibold">{topic.title}</h2>
									<p class="mt-2 text-sm leading-6 text-slate-400">{topic.description}</p>
								</div>

								<button
									type="button"
									on:click={() => toggleFavorite(topic.id)}
									class="rounded-xl border border-white/10 px-3 py-2 text-lg leading-none text-amber-300 transition hover:bg-white/5"
									aria-label="Toggle favorite"
								>
									{topic.favorite ? '★' : '☆'}
								</button>
							</div>

							<div class="mt-4 flex items-center justify-between">
								<span class={`rounded-full px-2.5 py-1 text-xs font-medium ${badge(topic.level)}`}>
									{topic.level}
								</span>
								<span class="text-xs text-slate-400">{topic.lessons} lessons</span>
							</div>

							<div class="mt-4 h-2 overflow-hidden rounded-full bg-slate-800">
								<div
									class={`h-2 rounded-full ${progressTone(topic.progress)}`}
									style={`width: ${topic.progress}%`}
								></div>
							</div>

							<div class="mt-2 flex items-center justify-between">
								<p class="text-xs text-slate-400">{topic.progress}% complete</p>
								<div class="flex flex-wrap justify-end gap-1">
									{#each topic.tags.slice(0, 2) as tag}
										<span class="rounded-full bg-white/5 px-2 py-1 text-[11px] text-slate-300"
											>#{tag}</span
										>
									{/each}
								</div>
							</div>

							<div class="mt-5 flex gap-2">
								<button
									type="button"
									on:click={() => selectTopic(topic.id)}
									class="flex-1 rounded-2xl bg-cyan-400 px-4 py-2.5 text-sm font-semibold text-slate-950 transition hover:bg-cyan-300"
								>
									Open topic
								</button>
								<button
									type="button"
									on:click={() => practice(topic.id)}
									class="rounded-2xl border border-white/10 px-4 py-2.5 text-sm font-medium transition hover:bg-white/5"
								>
									+10%
								</button>
							</div>
						</article>
					{/each}

					{#if !filteredTopics.length}
						<div
							class="rounded-3xl border border-dashed border-white/10 bg-slate-950/60 p-10 text-center text-slate-400 md:col-span-2"
						>
							<p class="text-lg font-semibold text-slate-200">No se encontraron temas</p>
							<p class="mt-2 text-sm">Prueba con otra búsqueda o cambia los filtros.</p>
						</div>
					{/if}
				</div>
			</section>

			<aside
				class="rounded-3xl border border-white/10 bg-slate-900/70 p-6 shadow-2xl shadow-black/30 backdrop-blur lg:sticky lg:top-6 lg:h-fit"
			>
				{#if selectedTopic}
					<div class="flex items-start justify-between gap-3">
						<div>
							<p class="text-sm text-cyan-300">Tema seleccionado</p>
							<h2 class="mt-1 text-2xl font-bold">{selectedTopic.title}</h2>
						</div>
						<span
							class={`rounded-full px-3 py-1 text-xs font-medium ${badge(selectedTopic.level)}`}
						>
							{selectedTopic.level}
						</span>
					</div>

					<p class="mt-4 text-sm leading-6 text-slate-300">{selectedTopic.description}</p>

					<div class="mt-6 grid grid-cols-2 gap-3">
						<div class="rounded-2xl border border-white/10 bg-slate-950/70 p-4">
							<p class="text-xs text-slate-400">Lessons</p>
							<p class="mt-1 text-xl font-semibold">{selectedTopic.lessons}</p>
						</div>
						<div class="rounded-2xl border border-white/10 bg-slate-950/70 p-4">
							<p class="text-xs text-slate-400">Progress</p>
							<p class="mt-1 text-xl font-semibold">{selectedTopic.progress}%</p>
						</div>
					</div>

					<div class="mt-6">
						<div class="mb-2 flex items-center justify-between text-sm">
							<span class="text-slate-400">Avance</span>
							<span class="font-medium">{selectedTopic.progress}%</span>
						</div>
						<div class="h-3 overflow-hidden rounded-full bg-slate-800">
							<div
								class={`h-3 rounded-full ${progressTone(selectedTopic.progress)}`}
								style={`width: ${selectedTopic.progress}%`}
							></div>
						</div>
					</div>

					<div class="mt-6 flex flex-wrap gap-2">
						{#each selectedTopic.tags as tag}
							<span
								class="rounded-full border border-white/10 bg-white/5 px-3 py-1 text-xs text-slate-300"
							>
								#{tag}
							</span>
						{/each}
					</div>

					<div class="mt-6 flex flex-wrap gap-3">
						<button
							type="button"
							on:click={() => practice(selectedTopic.id)}
							class="rounded-2xl bg-cyan-400 px-4 py-3 text-sm font-semibold text-slate-950 transition hover:bg-cyan-300"
						>
							Practice +10%
						</button>
						<button
							type="button"
							on:click={() => toggleFavorite(selectedTopic.id)}
							class="rounded-2xl border border-white/10 px-4 py-3 text-sm transition hover:bg-white/5"
						>
							{selectedTopic.favorite ? 'Unfavorite' : 'Favorite'}
						</button>
						<button
							type="button"
							on:click={() => reset(selectedTopic.id)}
							class="rounded-2xl border border-rose-400/20 bg-rose-500/10 px-4 py-3 text-sm text-rose-200 transition hover:bg-rose-500/20"
						>
							Reset
						</button>
					</div>
				{:else}
					<div
						class="flex min-h-[320px] items-center justify-center rounded-3xl border border-dashed border-white/10 bg-slate-950/50 p-8 text-center text-sm text-slate-400"
					>
						Selecciona un tema para ver el detalle.
					</div>
				{/if}
			</aside>
		</div>
	</div>
</div>
