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
    import {CONTRACT_ID, MY_ACCOUNT_ID1, MY_PRIVATE_KEY1} from "$stores/hederaStores.js";

    let fileVar;
    let files;
    let encryptedFile;
    let encryptedBlob;
    let password;
    let cid;
    let checksum = "Place Holder";

    $: if (files) {
        // Note that `files` is of type `FileList`, not an Array:
        // https://developer.mozilla.org/en-US/docs/Web/API/FileList
        console.log(files);

        for (const file of files) {
            console.log(`${file.name}: ${file.size} bytes`);
        }
    }

    // Encryption
    async function encryptFile() {
        const file = fileVar;
        password = generateId(2048); // Replace with your encryption password
        console.log(password);

        if (!file) {
            console.error("No file selected.");
            return;
        }

        const reader = await new FileReader();
        reader.onload = async () => {
            try {
                const fileData = reader.result;
                const wordArray = await CryptoJS.lib.WordArray.create(fileData);

                const encrypted = await CryptoJS.AES.encrypt(wordArray, password);
                encryptedBlob = await new Blob([encrypted.toString()], {
                    type: file.type, name: file.name
                });

                // Do further processing with the encrypted file
                console.log("Encrypted Blob:", encryptedBlob);
            } catch (error) {
                console.error("Encryption failed:", error);
            }
        };

        reader.onerror = (error) => {
            console.error("File reading error:", error);
        };

        await reader.readAsArrayBuffer(file);
    }


    // dec2hex :: Integer -> String
    // i.e. 0-255 -> '00'-'ff'
    function dec2hex(dec) {
        return dec.toString(16).padStart(2, "0")
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
            await encryptFile();
            await waitForElement();
        } else {
            console.log("No file selected");
        }
    }

    // Kinda of a hacky way to wait <- Need to find a better way
    function waitForElement() {
        if (typeof encryptedBlob !== "undefined") {
            encryptedFile = new File([encryptedBlob], fileVar.name, {
                type: fileVar.type, lastModified:
                fileVar.lastModified
            });
            console.log("Encrypted file:", encryptedFile);
            console.log("Uploading file...");
            uploadFile();
        } else {
            setTimeout(waitForElement, 250);
        }
    }


    async function uploadFile() {
        console.log("Encrypted file:", encryptedFile);

        const token = web3StorageToken

        if (!token) {
            return console.error('A token is needed. You can create one on https://web3.storage')
        }

        const storage = new Web3Storage({token});
        cid = await storage.put([encryptedFile]);
        console.log("CID:", cid);

        await uploadFileToHedera();
    }

    async function uploadFileToHedera() {
        const myAccountId = AccountId.fromString(MY_ACCOUNT_ID1);
        const myPrivateKey = PrivateKey.fromString(MY_PRIVATE_KEY1);
        const OEM = Client.forTestnet.setOperator(myAccountId, myPrivateKey);
        const contract = ContractId.fromString(CONTRACT_ID);
        const contractUploadUpdate = new ContractExecuteTransaction()
            .setContractId(contract)
            .setGas(1000000)
            .setFunction("addUpdate",
                new ContractFunctionParameters().addString(password).addString(checksum).addString(cid).addString("Version 2").addUint8(1));
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

