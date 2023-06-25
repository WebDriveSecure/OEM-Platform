<script>
    import {Web3Storage} from 'web3.storage'
    import {web3StorageToken} from "../stores/storageStores.js";
    import {CONTRACT_ID, MY_ACCOUNT_ID2, MY_PRIVATE_KEY2} from "../stores/hederaStores.js";
    import {
        AccountId,
        Client,
        ContractCallQuery,
        ContractExecuteTransaction,
        ContractFunctionParameters,
        ContractId, Key,
        PrivateKey
    } from "@hashgraph/sdk";
    import CryptoJS from "crypto-js";

    let key;
    let checksum;
    let cid;
    const MODEL_AId = AccountId.fromString(MY_ACCOUNT_ID2);
    const MODEL_AKey = PrivateKey.fromString(MY_PRIVATE_KEY2);
    const MODEL_A = Client.forTestnet().setOperator(MODEL_AId, MODEL_AKey);
    const contract = ContractId.fromString(CONTRACT_ID);

    async function submit() {
        const contractHasUpdate = new ContractCallQuery()
            .setContractId(contract)
            .setGas(100000)
            .setFunction("hasUpdate", new ContractFunctionParameters().addUint8(1));
        const contractHasUpdateResponse = await contractHasUpdate.execute(MODEL_A);
        const modelAHasUpdate = contractHasUpdateResponse.getBool(0);
        if (modelAHasUpdate) {
            console.log("Update available");
        } else {
            console.log("No update available");
            return;
        }
        // try {
        //     await grabFileInfo();
        //     await downloadFiles();
        //     await decryptFile();
        //     await waitForElement();
        // } catch (error) {
        //     const contractSendUpdateStatus = new ContractExecuteTransaction()
        //         .setContractId(contract)
        //         .setGas(1000000)
        //         .setFunction("updateUpdateStatus", new ContractFunctionParameters().addUint8(1).addBool(false));
        //     const contractSendUpdateStatusResponse = await contractSendUpdateStatus.execute(MODEL_A);
        //     const contractSendUpdateStatusReceipt = await contractSendUpdateStatusResponse.getReceipt(MODEL_A);
        //     console.log("Update Failed");
        //     throw new Error("Update Failed");
        // }
        // const contractSendUpdateStatus = new ContractExecuteTransaction()
        //     .setContractId(contract)
        //     .setGas(1000000)
        //     .setFunction("updateUpdateStatus", new ContractFunctionParameters().addUint8(1).addBool(true));
        // const contractSendUpdateStatusResponse = await contractSendUpdateStatus.execute(MODEL_A);
        // const contractSendUpdateStatusReceipt = await contractSendUpdateStatusResponse.getReceipt(MODEL_A);
        // console.log("Update Complete");
        await grabFileInfo();
        const encryptedFile = await fetchEncryptedFile(cid);
        console.log("Key:", key);
        const decryptedFile = await decryptFile(encryptedFile, key);
        console.log("File Decrypted");
        await downloadFile(decryptedFile);
    }

    // async function deriveKeyFromPassword(password) {
    //     const encoder = new TextEncoder();
    //     const passwordData = encoder.encode(password);
    //     const importedKey = await crypto.subtle.importKey('raw', passwordData, 'PBKDF2', false, ['deriveBits']);
    //     const keyMaterial = await crypto.subtle.deriveBits(
    //         {
    //             name: 'PBKDF2',
    //             salt: encoder.encode("Hello World"),
    //             iterations: 100000,
    //             hash: 'SHA-256',
    //         },
    //         importedKey,
    //         128 // 256-bit key length
    //     );
    //     const keyData = new Uint8Array(keyMaterial);
    //     return keyData.slice(0, 32); // Truncate to 256 bits (32 bytes)
    // }

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
        return keyData.slice(0, 16); // Truncate to 256 bits (32 bytes)
    }

    async function grabFileInfo() {
        const contractGetUpdate = new ContractCallQuery()
            .setContractId(contract)
            .setGas(100000)
            .setFunction("fetchUpdate", new ContractFunctionParameters().addUint8(1));
        const contractGetUpdateResponse = await contractGetUpdate.execute(MODEL_A);
        const modelAKey = contractGetUpdateResponse.getString(0);
        console.log("Update Key:", modelAKey)
        const modelAChecksum = contractGetUpdateResponse.getString(1);
        console.log("Update Checksum:", modelAChecksum)
        const modelACID = contractGetUpdateResponse.getString(2);
        console.log("Update CID:", modelACID)
        const modelAUpdateVersion = contractGetUpdateResponse.getString(3);
        console.log("Update Version:", modelAUpdateVersion);
        key = modelAKey;
        checksum = modelAChecksum;
        cid = modelACID;
    }

    // Function to download a file
    function downloadFile(blob, fileName) {
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = fileName;
        link.click();
    }

    // Function to decrypt a file using AES-GCM
    // async function decryptFile(file, key) {
    //     const algorithm = { name: 'AES-GCM', iv: file.slice(0, 12) };
    //     const decryptionKey = await generateEncryptionKey(key);
    //     const decryptedData = await window.crypto.subtle.decrypt(algorithm, decryptionKey, file.slice(12));
    //     return new Blob([decryptedData]);
    // }

    // Function to fetch the encrypted file from web3.storage
    async function fetchEncryptedFile(cid) {
        const endpoint = 'https://api.web3.storage';
        const response = await fetch(`${endpoint}/car/${cid}`);
        if (!response.ok) {
            throw new Error(`Failed to fetch file. Status: ${response.status}`);
        }
        return await response.blob();
    }


    /**
    async function decryptFile(file, key) {
        const encoder = new TextEncoder();
        const keyData = encoder.encode(key);
        const cryptoKey = await crypto.subtle.importKey('raw', keyData.slice(0, 16), 'AES-GCM', false, ['decrypt']);
        console.log("Crypto Key:", cryptoKey);
        const iv = new Uint8Array(await file.slice(0, 12).arrayBuffer());
        console.log("IV:", iv);
        const algorithm = { name: 'AES-GCM', iv };
        const encryptedData = await file.slice(12).arrayBuffer();
        console.log("Encrypted Data:", encryptedData);
        try {
            const decryptedData = await crypto.subtle.decrypt(algorithm, cryptoKey, encryptedData);
            console.log("Decrypted Data:", decryptedData);
            return new Blob([decryptedData]);
        } catch (error) {
            console.error("Decryption error:", error);
            throw error;
        }
    } **/

    async function decryptFile(file, key) {
        const iv = CryptoJS.lib.WordArray.create(file.slice(0, 12));
        const encryptedData = CryptoJS.lib.WordArray.create(file.slice(12));

        const decryptedData = CryptoJS.AES.decrypt({ ciphertext: encryptedData }, key, {
            iv: iv,
            padding: CryptoJS.pad.NoPadding,
            mode: CryptoJS.mode.GCM
        });

        const decryptedArrayBuffer = decryptedData.toString(CryptoJS.enc.Latin1).toArrayBuffer();
        return new Blob([decryptedArrayBuffer]);
    }

</script>

<!--    Select File     -->
<!--    Simple form that allows input of file   -->
<h1>Welcome to File Download</h1>
<h2>How to download files using HTML to website?</h2>
<div>
    <button on:click={submit}>Submit</button>
</div>


