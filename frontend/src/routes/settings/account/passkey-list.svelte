<script lang="ts">
	import { openConfirmDialog } from '$lib/components/confirm-dialog/';
	import { Button } from '$lib/components/ui/button';
	import { Separator } from '$lib/components/ui/separator';
	import WebauthnService from '$lib/services/webauthn-service';
	import type { Passkey } from '$lib/types/passkey.type';
	import { axiosErrorToast } from '$lib/utils/error-util';
	import { LucideKeyRound, LucidePencil, LucideTrash } from 'lucide-svelte';
	import { toast } from 'svelte-sonner';
	import RenamePasskeyModal from './rename-passkey-modal.svelte';

	let { passkeys = $bindable() }: { passkeys: Passkey[] } = $props();

	const webauthnService = new WebauthnService();

	let passkeyToRename: Passkey | null = $state(null);

	async function deletePasskey(passkey: Passkey) {
		openConfirmDialog({
			title: `Delete ${passkey.name}`,
			message: 'Are you sure you want to delete this passkey?',
			confirm: {
				label: 'Delete',
				destructive: true,
				action: async () => {
					try {
						await webauthnService.removeCredential(passkey.id);
						passkeys = await webauthnService.listCredentials();
						toast.success('Passkey deleted successfully');
					} catch (e) {
						axiosErrorToast(e);
					}
				}
			}
		});
	}
</script>

<div class="flex flex-col">
	{#each passkeys as passkey, i}
		<div class="flex justify-between">
			<div class="flex items-center">
				<LucideKeyRound class="mr-4 inline h-6 w-6" />
				<div>
					<p>{passkey.name}</p>
					<p class="text-xs text-muted-foreground">
						Added on {new Date(passkey.createdAt).toLocaleDateString()}
					</p>
				</div>
			</div>
			<div>
				<Button
					on:click={() => (passkeyToRename = passkey)}
					size="sm"
					variant="outline"
					aria-label="Rename"><LucidePencil class="h-3 w-3" /></Button
				>
				<Button
					on:click={() => deletePasskey(passkey)}
					size="sm"
					variant="outline"
					aria-label="Delete"><LucideTrash class="h-3 w-3 text-red-500" /></Button
				>
			</div>
		</div>
		{#if i !== passkeys.length - 1}
			<Separator class="my-2" />
		{/if}
	{/each}
</div>
<RenamePasskeyModal
	bind:passkey={passkeyToRename}
	callback={async () => (passkeys = await webauthnService.listCredentials())}
/>
