<script>
	/*
	Credits:
		https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto
		https://gist.github.com/danallison/3ec9d5314788b337b682
		https://stackoverflow.com/users/633183/thank-you
		https://stackoverflow.com/questions/1349404/generate-random-string-characters-in-javascript
		https://stackoverflow.com/questions/9267899/arraybuffer-to-base64-encoded-string
		https://stackoverflow.com/questions/21797299/convert-base64-string-to-arraybuffer
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

	function bufferToBase64 (buffer) {
		let binaryString = '';
		const bytesArray = new Uint8Array(buffer);
		const len = bytesArray.byteLength;
		for (let i = 0; i < len; i++) {
			binaryString += String.fromCharCode(bytesArray[i]);
		}
		const base64 = btoa(binaryString);
		return base64;
	}

	function base64Tobuffer(base64) {
		const binaryString = atob(base64);
		const len = binaryString.length;
		const bytesArray = new Uint8Array(len);
		for (let i = 0; i < len; i++) {
			bytesArray[i] = binaryString.charCodeAt(i);
		}
		return bytesArray.buffer;
	}

	async function encrypt () {
		const paddedSecret = secret + '='.repeat(maxSecretLength - secret.length);
		const encoder = new TextEncoder();
		const encoded = encoder.encode(paddedSecret);
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
		const dec = new TextDecoder();
		const encryptedObject = JSON.parse(fileContent);
		const counter = base64Tobuffer(encryptedObject.counter);
		const encrypted = base64Tobuffer(encryptedObject.encrypted);
		try {
			const decrypted = await window.crypto.subtle.decrypt(
				{
					name: "AES-CTR",
					counter,
					length: 64
				},
				key,
				encrypted,
			);
			const paddedDecryptedString = dec.decode(decrypted);
			decryptedString = paddedDecryptedString.replace(/=+$/,'');
		} catch (err) {
			console.error(err);
		}
	}

	function download () {
		const blob = new Blob([fileContent], { type: 'application/json' });

		const a = document.createElement('a');

		a.style.display = "none";
		a.download = 'encrypted.json';
		a.href = URL.createObjectURL(blob);
		a.dataset.downloadurl = ['application/json', a.download, a.href].join(':');

		document.body.appendChild(a);
		a.click();
		document.body.removeChild(a);
		setTimeout(function() { URL.revokeObjectURL(a.href); }, 1500);
	}

	$: if (counter) {
		encryptedObject = {
			counter: bufferToBase64(counter),
			encrypted: bufferToBase64(encrypted),
		}
		fileContent = JSON.stringify(encryptedObject);
	}

</script>

<main>
	<h1>Web AES</h1>

	<nav>
		<ul>
			<li on:click="{()=>action="encrypt"}">Cifrar</li>
			<li on:click="{()=>action="decrypt"}">Descifrar</li>
		</ul>
	</nav>

	{#if action==="encrypt"}

	<h2>Cifrar</h2>

	<h3>Frase de cifrado:</h3>
	<input type="text" bind:value="{passphrase}" on:input="{derivateKey}"/>

	<h3>Texto a cifrar:</h3>
	<textarea disabled="{key ? false : true}" maxlength="446" bind:value={secret} on:input={encrypt}></textarea>

	<button disabled="{passphrase && secret ? false : true}" on:click="{download}">Descargar fichero cifrado</button>

	{:else if action==="decrypt"}

	<h2>Descifrar</h2>

	<h3>Datos cifrados</h3>
	<p>Selecciona el fichero o pega el texto.</p>
	<textarea bind:value="{fileContent}"></textarea>
	<button>Selecciona el fichero</button>

	<h3>Frase de cifrado:</h3>
	<input type="text" bind:value="{passphrase}"/>
	<button on:click="{decrypt}">Descifrar</button>

	<h3>Datos descifrados</h3>
	<textarea disabled="true" bind:value="{decryptedString}"></textarea>

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