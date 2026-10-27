<script lang="ts">
	import type { PreAuthKey } from '$lib/common/types';
	import RawMdiClipboard from '~icons/mdi/clipboard';
	import { getToastStore, popup, type PopupSettings } from '@skeletonlabs/skeleton';
	import { copyToClipboard, toastSuccess } from '$lib/common/funcs';
	import Delete from '$lib/parts/Delete.svelte';
	import { deletePreAuthKey, expirePreAuthKey, getPreAuthKeys } from '$lib/common/api';
	import { App } from '$lib/States.svelte';
	import { onMount } from 'svelte';

	type UserListPreAuthKeyProps = {
		preAuthKey: PreAuthKey,
	}
	let { preAuthKey }: UserListPreAuthKeyProps = $props()

	const toastStore = getToastStore();
	let pakIsExpired = $derived(isExpired(preAuthKey))

	function isExpired(preAuthKey: PreAuthKey): boolean {
		return new Date() > new Date(preAuthKey.expiration);
	}

	onMount(()=>{
		const interval = setInterval(() => {
			pakIsExpired = isExpired(preAuthKey)
		}, 1000)

		return () => {
			clearInterval(interval)
		}
	})
</script>

<div class="col-span-12 flex w-full items-center justify-between">
	<div class="flex items-center gap-3">
		<button
			class="font-mono flex items-center border-2 border-dashed w-auto py-1.5 px-2 mr-3 border-slate-300 dark:border-slate-700 whitespace-nowrap"
			onclick={() => copyToClipboard(preAuthKey.key, toastStore)}
		>
			<span class="mr-2">
				<RawMdiClipboard />
			</span>
			{preAuthKey.key.substring(11, 23)}
		</button>
		<div class="flex items-center flex-row gap-1 lg:gap-2">
			<span
				class="badge badge-glass {preAuthKey.used
					? 'variant-ghost-success'
					: 'variant-flat opacity-50'}"
			>
				Used
			</span>
			<span
				class="badge badge-glass {pakIsExpired
					? 'variant-ghost-error'
					: 'variant-flat opacity-50'}"
			>
				Expired
			</span>
		</div>
		<div class="flex items-center flex-row gap-1 lg:gap-2">
			<span
				class="badge badge-glass {preAuthKey.ephemeral
					? 'variant-ghost-secondary'
					: 'variant-flat opacity-50'}"
			>
				Ephemeral
			</span>
			<span
				class="badge badge-glass {preAuthKey.reusable
					? 'variant-ghost-tertiary'
					: 'variant-flat opacity-50'}"
			>
				Reusable
			</span>
			{#if preAuthKey.ipv4 || preAuthKey.ipv6}
				<span class="badge badge-glass variant-ghost-primary" title="Fixed IP for the node that registers with this key">
					{[preAuthKey.ipv4, preAuthKey.ipv6].filter((ip) => !!ip).join(', ')}
				</span>
			{/if}
		</div>
	</div>
	<div class="flex">
		<span class="mr-2">
			<Delete
				func={async () => {
					try{
						await deletePreAuthKey(preAuthKey);
					} catch(e) {
						toastSuccess('Error Deleting PreAuth Key: ' + e, toastStore);
						return;
					}
					toastSuccess('PreAuth Key Deleted', toastStore);
					App.removeValue(App.preAuthKeys, preAuthKey);
					const keys = await getPreAuthKeys();
					App.preAuthKeys.value = keys;
				}}
			/>
		</span>
	</div>
</div>
