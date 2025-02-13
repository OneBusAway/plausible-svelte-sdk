<script module lang="ts">
	interface PlausibleTracker {
		(event: string, options?: any): void;
	}

	interface PlausibleWindow extends Window {
		plausible: PlausibleTracker;
	}

	declare let window: PlausibleWindow;

	const plausible: PlausibleTracker = (event, options) => window.plausible(event, options);
</script>

<script lang="ts">
	import { onMount } from 'svelte';
	import { dev } from '$app/environment';
	import { page } from '$app/stores';
	import { pa } from '$lib/store.js';

	onMount(() => {
		pa.subscribe((events) => {
			let next = events.length && events.shift();

			while (next) {
				const { event, data } = next;
				plausible(event, data);
				next = events.shift();
			}
		});
	});

	interface Props {
		/**
		 * The API host.
		 *
		 * @defaultValue 'https://plausible.io'
		 */
		apiHost?: string;
		/**
		 * Compatibility mode for tracking users on Internet Explorer.
		 *
		 * @defaultValue false
		 */
		compat?: boolean;
		/**
		 * The domain name(s) of the website(s) to track.
		 *
		 * @defaultValue current hostname.
		 */
		domain?: string | string[];
		/**
		 * Enable analytics.
		 *
		 * @defaultValue `true` in production mode, `false` in development mode.
		 */
		enabled?: any;
		/**
		 * Automatically track file downloads
		 * (Requires manual goal configuration on Plausible.)
		 * @defaultValue `false`
		 */
		fileDownloads?: boolean;
		/**
		 * Automatically follow frontend navigation when using hash-based routing.
		 *
		 * @defaultValue `false`
		 */
		hash?: boolean;
		/**
		 * Allow analytics to track on localhost (useful in hybrid apps).
		 * @defaultValue `false`, unless `enabled` is `true` in development mode.
		 */
		local?: boolean;
		/**
		 * Automatically track clicks on outbound links from your website.
		 * (Requires manual goal configuration on Plausible.)
		 * @defaultValue `false`
		 */
		outboundLinks?: boolean;
	}

	let {
		apiHost = 'https://plausible.io',
		compat = false,
		domain = $page.url.hostname,
		enabled = !dev,
		fileDownloads = false,
		hash = false,
		local = enabled && dev,
		outboundLinks = false
	}: Props = $props();

	let api = $derived(`${apiHost}/api/event`);
	let src = $derived(
		[
			`${apiHost}/js/script`,
			compat ? 'compat' : undefined,
			fileDownloads ? 'file-downloads' : undefined,
			hash ? 'hash' : undefined,
			local ? 'local' : undefined,
			outboundLinks ? 'outbound-links' : undefined,
			'js'
		]
			.filter(Boolean)
			.join('.')
	);
</script>

<svelte:head>
	{#if enabled}
		<script data-api={api} data-domain={domain.toString()} defer {src}></script>
		<script>
			window.plausible =
				window.plausible ||
				function () {
					(window.plausible.q = window.plausible.q || []).push(arguments);
				};
		</script>
	{/if}
</svelte:head>
