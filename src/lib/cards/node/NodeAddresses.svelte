<script lang="ts">
	import CardListEntry from '../CardListEntry.svelte';
	import type { Node } from '$lib/common/types';
	import { fade } from 'svelte/transition';
	import RawMdiPencil from '~icons/mdi/pencil';
	import RawMdiCheckCircleOutline from '~icons/mdi/check-circle-outline';
	import RawMdiCloseCircleOutline from '~icons/mdi/close-circle-outline';
	import { setNodeIPs } from '$lib/common/api';
	import { toastError, toastSuccess, focus } from '$lib/common/funcs';
	import { getToastStore } from '@skeletonlabs/skeleton';
	import { debug } from '$lib/common/debug';

	type NodeAddressesProps = {
		node: Node,
	}

	let { node }: NodeAddressesProps = $props()

	const isV4 = (s: string) => /^\d+\.\d+\.\d+\.\d+$/.test(s);
	const currentV4 = $derived(node.ipAddresses.find(isV4) ?? '');
	const currentV6 = $derived(node.ipAddresses.find((s) => s.includes(':')) ?? '');

	let newV4 = $state('');
	let newV6 = $state('');
	let showEdit = $state(false);
	let disableEdit = $state(false);

	const fadeDuration = 200;
	const ToastStore = getToastStore();
</script>

<CardListEntry title="IP Addresses:" top>
	<div class="grid text-right overlap-children">
		{#if showEdit}
			<div
				in:fade={{ delay: fadeDuration + 50, duration: fadeDuration }}
				out:fade={{ duration: fadeDuration }}
			>
				<form class="grid grid-cols-1 gap-1">
					<input
						id={`setip-v4-${node.id}`}
						class="input text-sm rounded-md py-0 pr-2"
						placeholder="IPv4"
						bind:value={newV4}
						disabled={disableEdit}
						use:focus
					/>
					<input
						class="input text-sm rounded-md py-0 pr-2"
						placeholder="IPv6"
						bind:value={newV6}
						disabled={disableEdit}
					/>
					<div class="flex justify-end">
						<button
							type="submit"
							class="btn-sm btn-icon-sm"
							disabled={disableEdit}
							onclick={async (event?: Event) => {
								event?.preventDefault();
								const v4 = newV4.trim() === currentV4 ? '' : newV4.trim();
								const v6 = newV6.trim() === currentV6 ? '' : newV6.trim();
								if (v4 === '' && v6 === '') {
									showEdit = false;
									return;
								}
								disableEdit = true;
								try {
									const n = await setNodeIPs(node, v4, v6);
									toastSuccess(`Node "${n.givenName}" now has ${n.ipAddresses.join(', ')}`, ToastStore);
									showEdit = false;
								} catch (error) {
									if (error instanceof Error) {
										toastError('', ToastStore, error);
									} else {
										debug(error);
									}
									focus(document.getElementById(`setip-v4-${node.id}`));
								} finally {
									disableEdit = false;
								}
							}}
						>
							<RawMdiCheckCircleOutline />
						</button>
						<button
							type="button"
							class="btn-sm btn-icon-sm"
							disabled={disableEdit}
							onclick={() => { showEdit = false; }}
						>
							<RawMdiCloseCircleOutline />
						</button>
					</div>
				</form>
			</div>
		{:else}
			<div
				in:fade={{ delay: fadeDuration + 50, duration: fadeDuration }}
				out:fade={{ duration: fadeDuration }}
			>
				<div class="flex items-start justify-end">
					<div class="grid grid-cols-1">
						{#each node.ipAddresses as ipAddress}
							<div>{ipAddress}</div>
						{/each}
					</div>
					<button
						class="btn-sm btn-icon-sm"
						title="Set IP"
						onclick={() => {
							newV4 = currentV4;
							newV6 = currentV6;
							showEdit = true;
						}}
					>
						<RawMdiPencil />
					</button>
				</div>
			</div>
		{/if}
	</div>
</CardListEntry>
