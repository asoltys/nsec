<script>
	import { onMount } from 'svelte';
	import { bytesToHex, hexToBytes } from '@noble/hashes/utils';
	import { HDKey } from '@scure/bip32';
	import { entropyToMnemonic, mnemonicToSeed } from '@scure/bip39';
	import { wordlist } from '@scure/bip39/wordlists/english';
	import { getPublicKey, nip19 } from 'nostr-tools';

	const version = 'bitcoin';
	export const versions = {
		bitcoin: {
			private: 0x0488ade4,
			public: 0x0488b21e
		},
		regtest: {
			private: 0x04358394,
			public: 0x043587cf
		}
	}[version];

	let mnemonic = $state(
		'leader monkey parrot ring guide accident before fence cannon height naive bean'
	);

	let nsec = $state();
	let npub = $state();

	let getNsec = async () => {
		try {
			const seed = await mnemonicToSeed(mnemonic);
			const master = HDKey.fromMasterSeed(seed, versions);
			const child = master.derive("m/44'/1237'/0'/0/0");
			const pubkey = child.publicKey;
			const sk = child.privateKey;
			const pk = getPublicKey(sk);

			nsec = nip19.nsecEncode(sk);
			npub = nip19.npubEncode(pk);
		} catch (e) {
			nsec = e.message;
		}
	};
</script>

<div style="width: 100%; display: flex; justify-content: center;">
	<div>
		<div>
			<textarea placeholder="Mnemonic" bind:value={mnemonic} rows={8}></textarea>
		</div>
		{#if nsec}
			<div style="font-size: 2em; width: 600px; padding: 20px; word-break: break-all;">{nsec}</div>
			<div style="font-size: 2em; width: 600px; padding: 20px; word-break: break-all">{npub}</div>
		{/if}
		<div style="@keyframes: linear 0%">
			<button type="submit" style="width:100%; color: white;" class="grad1" onclick={getNsec}
				>Get Nsec</button
			>
		</div>
	</div>
</div>

<style>
	textarea,
	button {
		font-size: 2em;
		width: 600px;
		padding: 20px;
	}

	.grad1 {
		background: black;
	}
</style>
