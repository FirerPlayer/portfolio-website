<script lang="ts">
	import { AtSymbol, Cube, Home, Identification } from 'svelte-heros-v2';
	import { onMount } from 'svelte/internal';
	import anime from 'animejs';
	import { currentPage } from '$lib/stores';
	import { goto } from '$app/navigation';

	import { page } from '$app/stores';
	let mainDiv: Element;
	let active: Element;
	let cloud: Element;

	const moveCloud = (el: Element) => {
		const rect = el.getBoundingClientRect();
		anime({
			targets: cloud,
			width: rect.width,
			top: rect.bottom + 3,
			left: rect.left,
			easing: 'spring',
			duration: 500,
			endDelay: 300
		});
	};
	function handleClick(ev: Event) {
		let value = (ev.target as HTMLButtonElement).value;
		goto(value === 'home' ? '/' : value);
		active = ev.target as HTMLButtonElement;
	}

	function handleHover(ev: Event) {
		moveCloud(ev.target as Element);
	}

	export function updateActive(btns: HTMLButtonElement[]) {
		let route = $page.route.id?.replace('/', '') as string;
		btns.forEach((el) => {
			if (el.value === $currentPage || el.value === route) {
				active = el;
			}
		});
	}

	onMount(() => {
		const observer = new IntersectionObserver(
			([entry]) => {
				if (entry.isIntersecting) {
					const btns = Array.from(mainDiv.children) as HTMLButtonElement[];
					if (!active) {
						updateActive(btns);
					}

					btns.forEach((el) => {
						el.addEventListener('mouseover', handleHover);
						el.addEventListener('focusin', handleHover);
						mainDiv.addEventListener('mouseout', () => moveCloud(active));
						mainDiv.addEventListener('focusout', () => moveCloud(active));
					});
					moveCloud(active);
					// console.log('nav is visible');
					// execute a ação que você deseja quando o elemento estiver visível
					// por exemplo, adicione uma classe CSS ou faça uma animação

					observer.disconnect();
				}
			},
			{ threshold: 1 }
		);

		observer.observe(mainDiv);
	});

	const btnClass = `flex flex-col md:flex-row gap-px md:gap-1 p-0 items-center`;
</script>

<svelte:window
	on:resize={() => {
		moveCloud(active);
	}}
/>
<div class="">
	<div
		bind:this={mainDiv}
		class="w-full absolute top-0 z-10 md:py-3 md:pl-5 flex flex-row items-center justify-evenly md:justify-start pr-1 gap-4"
	>
		<button aria-label="home" value="home" on:click={handleClick} class={btnClass}>
			<Home size="100%" class="w-6 pointer-events-none" tabindex="-1" />
			Home
		</button>
		<button aria-label="about" value="about" on:click={handleClick} class={btnClass}>
			<Identification tabindex="-1" size="100%" class="w-6 pointer-events-none" />
			About
		</button>
		<button aria-label="projects" value="projects" on:click={handleClick} class={btnClass}>
			<Cube class="w-6 pointer-events-none" tabindex="-1" size="100%" />
			Projects
		</button>
		<button aria-label="contact" value="contact" on:click={handleClick} class={btnClass}>
			<AtSymbol tabindex="-1" size="100%" class="w-6 pointer-events-none" />
			Contact
		</button>
	</div>
	<div bind:this={cloud} class="absolute left-0 z-0 h-1 bg-default" />
</div>
