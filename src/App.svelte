<script>
	/*
		From: Mozilla Developer Network Web Docs
		- https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/encrypt#rsa-oaep_2
		- https://github.com/mdn/dom-examples/blob/master/web-crypto/encrypt-decrypt/rsa-oaep.js
	*/

		/*
		Fetch the ciphertext and decrypt it.
		Write the decrypted message into the "Decrypted" box.
		
		async function decryptMessage(key) {
		  let decrypted = await window.crypto.subtle.decrypt(
			{
			  name: "RSA-OAEP"
			},
			key,
			ciphertext
		  );
	  
		  let dec = new TextDecoder();
		  const decryptedValue = document.querySelector(".rsa-oaep .decrypted-value");
		  decryptedValue.classList.add('fade-in');
		  decryptedValue.addEventListener('animationend', () => {
			decryptedValue.classList.remove('fade-in');
		  });
		  decryptedValue.textContent = dec.decode(decrypted);
		}
		*/
	/*
	End of Mozilla Developer Network Web Docs content.
	*/

	var action = "encrypt";
	var key, privateKey, secret, encrypted;

	function generateKey () {
		// Fragment taken from  Mozilla Developer Network Web Docs
		window.crypto.subtle.generateKey(
		  {
		  name: "RSA-OAEP",
		  modulusLength: 4096,
		  publicExponent: new Uint8Array([1, 0, 1]),
		  hash: "SHA-256",
		  },
		  true,
		  ["encrypt", "decrypt"]
		)
        // Fragment end
		.then(
			(k)=>{
				key = k;
				crypto.subtle.exportKey('pkcs8',key.privateKey).then(
					(p)=>{
						privateKey = `-----BEGIN PRIVATE KEY-----\n${btoa(String.fromCharCode.apply(null, new Uint8Array(p)))}\n-----END PRIVATE KEY-----`;
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
					name: "RSA-OAEP"
				},
				key.publicKey,
				encoded
			);
		} catch (err) {
			console.error(err);
		}
		encrypted = arrayBufferToBase64(cipher);
	}

	/**
	 * https://gist.github.com/danallison/3ec9d5314788b337b682
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

	generateKey();

	$:{

	}

</script>

<main>
	<h1>Web RSA</h1>

	<h2>Cifrar</h2>


	{#if action==="encrypt"}


	<h3>Texto a cifrar:</h3>
	<textarea disabled="{privateKey ? false : true}" maxlength="446" bind:value={secret} on:input={encrypt}></textarea>

	<h3>Clave secreta:</h3>
	<textarea disabled bind:value={privateKey}></textarea>
	<button>Download</button>


	<h3>Información cifrada:</h3>
	<textarea disabled bind:value="{encrypted}"></textarea>
	<button>Download</button>

	<a href="{`data:text/plain;charset=utf-8,${encodeURIComponent(encrypted)}`}" download="encripted.txt">text file</a>

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