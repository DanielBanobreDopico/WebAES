<script>
	/*
	Credits:
		Mozilla Developer Network Web Docs
			- https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto
		Dan Allison
			- https://gist.github.com/danallison/3ec9d5314788b337b682
	*/

	let action = "encrypt";
	let passphrase, salt, key, exportedKey, counter, secret, encrypted;

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
			salt: salt,
			"iterations": 100000,
			"hash": "SHA-512"
			},
			await keySeed(),
			{ "name": "AES-CTR", "length": 256},
			true,
			[ "encrypt", "decrypt" ]
		);
		exportedKey = await crypto.subtle.exportKey('jwk',key);
		encrypt();
	}

	function generateKey () {
		window.crypto.subtle.generateKey(
		  {
			name: "AES-CTR",
			length: 256,
		  },
		  true,
		  ["encrypt", "decrypt"]
		)
		.then(
			(k)=>{
				key = k;
				crypto.subtle.exportKey('jwk',key).then(
					(p)=>{
						console.log(JSON.stringify(p))
						exportedKey = JSON.stringify(p);
					}
				)
			}
		)
	}

	function arrayBufferToBase64( buffer ) {
		let binary = '';
		const bytes = new Uint8Array( buffer );
		const len = bytes.byteLength;
		for (var i = 0; i < len; i++) {
			binary += String.fromCharCode( bytes[i] );
		}
		return window.btoa( binary );
	}
	
	async function encrypt () {
		const encoder = new TextEncoder();
		const encoded = encoder.encode(secret);
		let cipher;
		try {
			cipher = await window.crypto.subtle.encrypt(
				{
					name: "AES-CTR",
					counter,
					length: 64
				},
				key,
				encoded
			);
		} catch (err) {
			console.error(err);
		}
		encrypted = arrayBufferToBase64(cipher);
	}

	/**
	 * 	<a href="{`data:text/plain;charset=utf-8,${encodeURIComponent(encrypted)}`}" download="encripted.txt">text file</a>
	 */
	function downloadString(text, fileType, fileName) {
		var blob = new Blob([text], { type: fileType });

		var a = document.createElement('a');
		a.download = fileName;
		a.href = URL.createObjectURL(blob);
		a.dataset.downloadurl = [fileType, a.download, a.href].join(':');
		a.style.display = "none";
		document.body.appendChild(a);
		a.click();
		document.body.removeChild(a);
		setTimeout(function() { URL.revokeObjectURL(a.href); }, 1500);
	}

	salt = crypto.getRandomValues(new Uint8Array(16));
	counter = crypto.getRandomValues(new Uint8Array(16));
	generateKey();

	$:{
		
	}

</script>

<main>
	<h1>Web AES</h1>

	<h2>Cifrar</h2>

	{#if action==="encrypt"}

	<h3>Texto a cifrar:</h3>
	<textarea disabled="{key ? false : true}" maxlength="446" bind:value={secret} on:input={encrypt}></textarea>

	<h3>Frase de paso:</h3>
	<input type="text" bind:value="{passphrase}" on:input="{derivateKey}"/>

	<h3>Información cifrada:</h3>
	<textarea disabled bind:value="{encrypted}"></textarea>
	<button>Descargar ficheros de clave y texto cifrado</button>

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