<script>
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
    let model = "";
    let version = "";
    let models = [
        {id: 1, text: `Model A`},
        {id: 2, text: `Model B`},
        {id: 3, text: `Model C`}
    ];

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

            reader.onload = function () {
                const wordArray = CryptoJS.lib.WordArray.create(reader.result);
                const checksum = CryptoJS.SHA256(wordArray).toString();
                resolve(checksum);
            };

            reader.onerror = function (error) {
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


    async function submit() {
        console.log("Submitted");
        if (files && files[0]) {
            fileVar = files[0];
            const checksum = await calculateChecksum(fileVar);
            console.log("Checksum:", checksum);
            const passwordData = generateId(20)
            // const password = await deriveKeyFromPassword(passwordData);
            console.log("Key:", passwordData);
            const encryptedFile = await encryptFile(fileVar, passwordData);
            const cid = await uploadFileToWeb3Storage(fileVar);
            console.log("CID:", cid);
            await uploadFileToHedera(checksum, passwordData, cid, model, version);
        } else {
            console.log("No file selected");
        }
    }

    // Function to encrypt a file using AES-GCM
    // async function encryptFile(file, key) {
    //     const algorithm = 'AES-GCM';
    //     const iv = generateRandomIV();
    //     const encryptionKey = await generateEncryptionKey(key);
    //     const cipher = window.crypto.subtle.encrypt({ name: algorithm, iv }, encryptionKey, file);
    //     const encryptedData = await cipher;
    //     const encryptedArray = new Uint8Array(encryptedData);
    //     const encryptedFile = new Blob([iv, encryptedArray]);
    //     return encryptedFile;
    // }

    // Web3.Storage upload function
    async function uploadFileToWeb3Storage(file) {
        const endpoint = 'https://api.web3.storage';

        const response = await fetch(`${endpoint}/upload`, {
            method: 'POST',
            headers: {
                Authorization: `Bearer ${web3StorageToken}`,
            },
            body: file,
        });

        const data = await response.json();
        return data.cid;
    }


    // Function to encrypt a file using a symmetric key
    async function encryptFile(file, key) {
        console.log(key);
        const fileData = await file.arrayBuffer();
        const encoder = new TextEncoder();
        const keyData = encoder.encode(key);
        const cryptoKey = await crypto.subtle.importKey('raw', keyData.slice(0, 16), 'AES-GCM', false, ['encrypt']);
        const iv = crypto.getRandomValues(new Uint8Array(12));
        const encryptedData = await crypto.subtle.encrypt({name: 'AES-GCM', iv}, cryptoKey, fileData);
        return new Blob([iv, encryptedData], {type: file.type, name: file.name});
    }


    async function uploadFileToHedera(checksum, key, cid, model, version) {
        const myAccountId = AccountId.fromString(MY_ACCOUNT_ID1);
        const myPrivateKey = PrivateKey.fromString(MY_PRIVATE_KEY1);
        const OEM = Client.forTestnet().setOperator(myAccountId, myPrivateKey);
        const contract = ContractId.fromString(CONTRACT_ID);
        const contractUploadUpdate = new ContractExecuteTransaction()
            .setContractId(contract)
            .setGas(1000000)
            .setFunction("addUpdate",
                new ContractFunctionParameters().addString(key).addString(checksum).addString(cid).addString(version).addUint8(model));
        const txResponse = await contractUploadUpdate.execute(OEM);
        const receipt = await txResponse.getReceipt(OEM);
        console.log("The transaction consensus status " + receipt.status.toString());
    }

</script>

<!--    Select File     -->
<!--    Simple form that allows input of file   -->
<h1>Welcome to WebDriveSecure</h1>
<h2>Use this client to help upload files securely</h2>

<form on:submit|preventDefault={submit}>
    <input
            bind:files
            id="file"
            type="file"
    />

    <select value={model} on:change={() => (version = '')}>
        {#each models as modelName}
            <option value={modelName}>
                {modelName.text}
            </option>
        {/each}
    </select>

    <input bind:value={version}/>

    <button disabled={!version} type="submit"> Submit</button>
</form>

<style>
    /* Style the input fields for the form so that each field is spaced out and everything is centered */
    input[type=text], input[type=password] {
        width: 100%;
        padding: 15px;
        margin: 5px 0 22px 0;
        display: inline-block;
        border: none;
        background: #f1f1f1;
    }

    /* Add a background color when the inputs get focus */
    input[type=text]:focus, input[type=password]:focus {
        background-color: #ddd;
        outline: none;
    }
    

</style>