<script>
	/*
	Credits:
		Mozilla Developer Network Web Docs
			- https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto
		Dan Allison
			- https://gist.github.com/danallison/3ec9d5314788b337b682
		Renato Mangini
			- https://developers.google.com/web/updates/2012/06/How-to-convert-ArrayBuffer-to-and-from-String
		https://stackoverflow.com/users/633183/thank-you
			- https://stackoverflow.com/questions/1349404/generate-random-string-characters-in-javascript
	*/

	const maxSecretLength = 446;
	let action = "encrypt";
	let passphrase, key, counter, encrypted, encryptedObject, decrypted, decryptedString, fileContent;
	let secret = '';

	function keySeed () {
		let enc = new TextEncoder();
		let seed = enc.encode(passphrase);
		return window.crypto.subtle.importKey(
			"raw",
			seed,
			"PBKDF2",
			false,
			["deriveBits", "deriveKey"]
 		);
	}

	async function derivateKey () {
		key = await window.crypto.subtle.deriveKey(
			{
			"name": "PBKDF2",
			salt: crypto.getRandomValues(new Uint8Array(16)),
			"iterations": 100000,
			"hash": "SHA-512"
			},
			await keySeed(),
			{ "name": "AES-CTR", "length": 256},
			true,
			[ "encrypt", "decrypt" ]
		);

		encrypt();
	}

	function ab2str(buf) {
		return String.fromCharCode.apply(null, new Uint16Array(buf));
	}

	function str2ab(str) {
		var buf = new ArrayBuffer(str.length*2); // 2 bytes for each char
		var bufView = new Uint16Array(buf);
		for (var i=0, strLen=str.length; i < strLen; i++) {
			bufView[i] = str.charCodeAt(i);
		}
		return buf;
	}

	async function encrypt () {
		const paddedSecret = secret + '='.repeat(maxSecretLength - secret.length);
		const encoder = new TextEncoder();
		const encoded = encoder.encode(JSON.stringify(paddedSecret));
		counter = crypto.getRandomValues(new Uint8Array(16));
		try {
			encrypted = await window.crypto.subtle.encrypt(
				{
					name: "AES-CTR",
					counter,
					length: 64
				},
				key,
				encoded,
			);
		} catch (err) {
			console.error(err);
		}
	}

	async function decrypt () {
		const encryptedObject = JSON.parse(fileContent);
		const counterValues = Object.values(encryptedObject.counter);
		const counterArray = new Uint8Array(counterValues);
		const counter = counterArray.buffer
		const encriptedValues = Object.values(encryptedObject.encrypted);
		const encryptedArray = new Uint8Array(encriptedValues);
		const encrypted = encryptedArray.buffer
		try {
			let decryptedBuffer = await window.crypto.subtle.decrypt(
				{
					name: "AES-CTR",
					counter,
					length: 64
				},
				key,
				encrypted,
			);
			decrypted = new Uint8Array(decryptedBuffer);
		} catch (err) {
			console.error(err);
		}
	}

	function download () {

		const blob = new Blob([encryptedObject], { type: 'application/json' });

		const a = document.createElement('a');

		a.download = 'encrypted.json';
		a.href = URL.createObjectURL(blob);
		a.dataset.downloadurl = ['application/json', a.download, a.href].join(':');
		a.style.display = "none";

		document.body.appendChild(a);
		a.click();

		document.body.removeChild(a);
		setTimeout(function() { URL.revokeObjectURL(a.href); }, 1500);
	}

	$: if (counter) {
		encryptedObject = {
				counter: new Uint8Array(counter),
				encrypted: new Uint8Array(encrypted),

			}
		fileContent = JSON.stringify(encryptedObject);
	}

	$: if (fileContent) {
		decrypt()
		let dec = new TextDecoder();
		let paddedDecryptedString = dec.decode(decrypted);
		decryptedString = paddedDecryptedString.replace(/=.*$/,'');
	}

	/**
	 * ArrayBuffer -> Uint8Array -> Blob -> btoa
	 * atob -> Uint8Array -> Blob -> ArrayBuffer
	*/
</script>

<main>
	<h1>Web AES</h1>

	<h2>Cifrar</h2>

	{#if action==="encrypt"}

	<h3>Frase de cifrado:</h3>
	<input type="text" bind:value="{passphrase}" on:input="{derivateKey}"/>

	<h3>Texto a cifrar:</h3>
	<textarea disabled="{key ? false : true}" maxlength="446" bind:value={secret} on:input={encrypt}></textarea>

	<h3>Información cifrada:</h3>
	<textarea disabled bind:value="{fileContent}"></textarea>
	<button on:click="{download}">Descargar fichero cifrado</button>
	<textarea disabled bind:value="{decryptedString}"></textarea>

	{:else if action==="decript"}

	<h2>Descifrar</h2>

	{/if}
</main>

<style>
	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>