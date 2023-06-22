<script>
    import {Web3Storage} from 'web3.storage'
    import {web3StorageToken} from "../stores/storageStores.js";
    import CryptoJS from 'crypto-js';
    import {
        AccountId,
        Client,
        ContractExecuteTransaction,
        ContractFunctionParameters,
        ContractId,
        PrivateKey
    } from "@hashgraph/sdk";
    import {CONTRACT_ID, MY_ACCOUNT_ID1, MY_PRIVATE_KEY1} from "../stores/hederaStores.js";

    let fileVar;
    let files;

    $: if (files) {
        // Note that `files` is of type `FileList`, not an Array:
        // https://developer.mozilla.org/en-US/docs/Web/API/FileList
        console.log(files);

        for (const file of files) {
            console.log(`${file.name}: ${file.size} bytes`);
        }
    }

    // dec2hex :: Integer -> String
    // i.e. 0-255 -> '00'-'ff'
    function dec2hex(dec) {
        return dec.toString(16).padStart(2, "0")
    }

    // Function to calculate the checksum of a File object
    function calculateChecksum(file) {
        return new Promise((resolve, reject) => {
            const reader = new FileReader();

            reader.onload = function() {
                const wordArray = CryptoJS.lib.WordArray.create(reader.result);
                const checksum = CryptoJS.SHA256(wordArray).toString();
                resolve(checksum);
            };

            reader.onerror = function(error) {
                reject(error);
            };

            reader.readAsArrayBuffer(file);
        });
    }


    // generateId :: Integer -> String
    function generateId(len) {
        var arr = new Uint8Array((len || 40) / 2)
        window.crypto.getRandomValues(arr)
        return Array.from(arr, dec2hex).join('')
    }

    // Function to derive an AES key from a password using PBKDF2
    async function deriveKeyFromPassword(password) {
        const encoder = new TextEncoder();
        const passwordData = encoder.encode(password);
        const importedKey = await crypto.subtle.importKey('raw', passwordData, 'PBKDF2', false, ['deriveBits']);
        const keyMaterial = await crypto.subtle.deriveBits(
            {
                name: 'PBKDF2',
                salt: encoder.encode("Hello World"),
                iterations: 100000,
                hash: 'SHA-256',
            },
            importedKey,
            128 // 256-bit key length
        );
        const keyData = new Uint8Array(keyMaterial);
        return keyData.slice(0, 32); // Truncate to 256 bits (32 bytes)
    }

    async function submit() {
        console.log("Submitted");
        if (files && files[0]) {
            fileVar = files[0];
            const checksum = await calculateChecksum(fileVar);
            console.log("Checksum:", checksum);
            const key = await deriveKeyFromPassword(generateId(20));
            console.log("Key:", key);
            const encryptedFile = await encryptFile(fileVar, key);
            const cid = await uploadFile(encryptedFile);
            console.log("CID:", cid);
            await uploadFileToHedera(checksum, key, cid);
        } else {
            console.log("No file selected");
        }
    }


    // Function to encrypt a file using a symmetric key
    async function encryptFile(file, key) {
        console.log(key);
        const fileData = await file.arrayBuffer();
        const cryptoKey = await crypto.subtle.importKey('raw', key, 'AES-GCM', false, ['encrypt']);
        const iv = crypto.getRandomValues(new Uint8Array(12));
        const encryptedData = await crypto.subtle.encrypt({ name: 'AES-GCM', iv }, cryptoKey, fileData);
        return new Blob([iv, encryptedData], { type: file.type, name: file.name });
    }


    async function uploadFile(encryptedFile) {
        console.log("Encrypted file:", encryptedFile);

        const token = web3StorageToken

        if (!token) {
            return console.error('A token is needed. You can create one on https://web3.storage')
        }

        const storage = await new Web3Storage({token});
        return await storage.put([encryptedFile], {
            wrapWithDirectory: false});
    }

    async function uploadFileToHedera(checksum, key, cid) {
        const myAccountId = AccountId.fromString(MY_ACCOUNT_ID1);
        const myPrivateKey = PrivateKey.fromString(MY_PRIVATE_KEY1);
        const OEM = Client.forTestnet().setOperator(myAccountId, myPrivateKey);
        const contract = ContractId.fromString(CONTRACT_ID);
        const contractUploadUpdate = new ContractExecuteTransaction()
            .setContractId(contract)
            .setGas(1000000)
            .setFunction("addUpdate",
                new ContractFunctionParameters().addString(key).addString(checksum).addString(cid).addString("Version 2").addUint8(1));
        const txResponse = await contractUploadUpdate.execute(OEM);
        const receipt = await txResponse.getReceipt(OEM);
        console.log("The transaction consensus status " + receipt.status.toString());
    }

</script>

<!--    Select File     -->
<!--    Simple form that allows input of file   -->
<h1>Welcome to File Upload</h1>
<h2>How to upload files using HTML to website?</h2>
<div>
    <input
            bind:files
            id="file"
            type="file"
    />
    <button on:click={submit}>Submit</button>
</div>

{#if files}
    <h2>Selected files:</h2>
    {#each Array.from(files) as file}
        <p>{file.name} ({file.size} bytes) </p>
    {/each}
{/if}

