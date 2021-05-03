<script>
	/*
		From: Mozilla Developer Network Web Docs
		- https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/encrypt#rsa-oaep_2
		- https://github.com/mdn/dom-examples/blob/master/web-crypto/encrypt-decrypt/rsa-oaep.js
	*/
		/*
		Store the calculated ciphertext here, so we can decrypt the message later.
		*/
		let ciphertext;
	  
		/*
		Fetch the contents of the "message" textbox, and encode it
		in a form we can use for the encrypt operation.
		*/
		function getMessageEncoding() {
		  const messageBox = document.querySelector("#rsa-oaep-message");
		  let message = messageBox.value;
		  let enc = new TextEncoder();
		  return enc.encode(message);
		}
	  
		/*
		Get the encoded message, encrypt it and display a representation
		of the ciphertext in the "Ciphertext" element.
		*/
		async function encryptMessage(key) {
		  let encoded = getMessageEncoding();
		  ciphertext = await window.crypto.subtle.encrypt(
			{
			  name: "RSA-OAEP"
			},
			key,
			encoded
		  );
	  
		  let buffer = new Uint8Array(ciphertext, 0, 5);
		  const ciphertextValue = document.querySelector(".rsa-oaep .ciphertext-value");
		  ciphertextValue.classList.add('fade-in');
		  ciphertextValue.addEventListener('animationend', () => {
			ciphertextValue.classList.remove('fade-in');
		  });
		  ciphertextValue.textContent = `${buffer}...[${ciphertext.byteLength} bytes total]`;
		}
	  
		/*
		Fetch the ciphertext and decrypt it.
		Write the decrypted message into the "Decrypted" box.
		*/
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
	/*
	End of Mozilla Developer Network Web Docs content.
	*/	  

	function generateKey () {
		// Fragment taken from  Mozilla Developer Network Web Docs
		window.crypto.subtle.generateKey(
		  {
		  name: "RSA-OAEP",
		  // Consider using a 4096-bit key for systems that require long-term security
		  modulusLength: 2048,
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
	
	/*
	.then((keyPair) => {
		  const encryptButton = document.querySelector(".rsa-oaep .encrypt-button");
		  encryptButton.addEventListener("click", () => {
			encryptMessage(keyPair.publicKey);
		  });
	  
		  const decryptButton = document.querySelector(".rsa-oaep .decrypt-button");
		  decryptButton.addEventListener("click", () => {
			decryptMessage(keyPair.privateKey);
		  });
		})
	*/

	var action = "encrypt"
	var key, privateKey;
	generateKey();
	$: console.log(privateKey)

</script>

<main>
	<h1>Web RSA</h1>
	{#if action==="encrypt"}
	<h2>Cifrar</h2>
	<p>Clave secreta:</p>
	<textarea disabled="false" bind:value="{privateKey}"></textarea>
	<p>Información cifrada:</p>
	<textarea disabled="false" bind:value="{ciphertext}"></textarea>
	{:else if action==="decript"}
	<h2>Descifrar</h2>
	{/if}
</main>

<style>
	main {
		text-align: center;
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