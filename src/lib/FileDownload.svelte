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
        const encryptedFile = await downloadFiles(cid);
        key = await deriveKeyFromPassword(key);
        console.log("Key:", key);
        const decryptedFile = await decryptFile(encryptedFile, key);
        console.log("Decrypted file:", decryptedFile);
        await downloadFile(decryptedFile);
    }

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


    async function downloadFiles(cid) {
        console.log("Downloading files");
        const token = web3StorageToken
        if (!token) {
            return console.error('A token is needed. You can create one on https://web3.storage')
        }
        const storage = new Web3Storage({token})
        const res = await storage.get(cid)
        const files = await res.files()
        for (const file of files) {
            console.log(`${file.cid} -- ${file.path} -- ${file.size}`)
        }

        const file = files[0];
        const stream = await file.stream();
        return await streamToBlob(stream);
    }

    async function streamToBlob(stream) {
        const reader = stream.getReader();
        const chunks = [];
        let length = 0;

        while (true) {
            const { done, value } = await reader.read();

            if (done) {
                break;
            }

            chunks.push(value);
            length += value.length;
        }

        return new Blob(chunks, { type: 'application/octet-stream' });
    }

    function downloadFile(fileObject) {
        const fileURL = URL.createObjectURL(fileObject);

        const link = document.createElement('a');
        link.href = fileURL;
        link.download = fileObject.name;
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);

        // Revoke the object URL to free up memory
        URL.revokeObjectURL(fileURL);
    }

    async function decryptFile(file, key) {
        const fileData = await file.arrayBuffer();
        const cryptoKey = await crypto.subtle.importKey('raw', key, 'AES-GCM', false, ['decrypt']);
        console.log("Key Created");
        const iv = await fileData.slice(0, 12);
        console.log("IV:", iv);
        const encryptedData = await fileData.slice(12);
        console.log("Encrypted Data:", encryptedData);
        const decryptedData = await crypto.subtle.decrypt({ name: 'AES-GCM', iv }, cryptoKey, encryptedData);
        console.log("Decrypted Data:", decryptedData);
        const decryptedFile = await new Blob([decryptedData], { type: file.type, name: file.name });
        console.log("Decrypted File:", decryptedFile);
        return decryptedFile;
    }

</script>

<!--    Select File     -->
<!--    Simple form that allows input of file   -->
<h1>Welcome to File Download</h1>
<h2>How to download files using HTML to website?</h2>
<div>
    <button on:click={submit}>Submit</button>
</div>


