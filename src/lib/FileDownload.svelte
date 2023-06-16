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
        ContractId,
        PrivateKey
    } from "@hashgraph/sdk";

    let decryptedBlob;
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
        await downloadFiles();
        await decryptFile();
        await waitForElement();
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



    // ToDo: Need to edit this function, implementation is a bit wack
    function waitForElement() {
        if (typeof decryptedBlob !== "undefined") {
            const decryptedFile = new File([decryptedBlob], encryptedFile.name, {
                type: encryptedFile.type, lastModified:
                encryptedFile.lastModified
            });
            console.log("Decrypted file:", decryptedFile);
            console.log("Downloading file...");
            downloadFile(decryptedFile, "Hello World.txt");
        } else {
            setTimeout(waitForElement, 250);
        }
    }


    async function downloadFiles() {
        console.log("Downloading files");

        const token = web3StorageToken

        if (!token) {
            return console.error('A token is needed. You can create one on https://web3.storage')
        }

        const storage = new Web3Storage({token})
        const res = await storage.get(cid)
        const files = await res.files()
        encryptedFile = files[0];
        for (const file of files) {
            console.log(`${file.cid} -- ${file.path} -- ${file.size}`)
            // download file
            // downloadFile(file, "Hello World.txt")
        }
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

    function decryptFile() {
        const password = key; // Replace with your encryption password

        if (!encryptedFile) {
            console.error("No encrypted file available.");
            return;
        }

        const reader = new FileReader();
        reader.onload = () => {
            try {
                const decrypted = CryptoJS.AES.decrypt(reader.result.toString(), key);//CryptoJS.AES.decrypt(reader.result, password);
                const decryptedData = decrypted.toString(CryptoJS.enc.Utf8);


                decryptedBlob = new Blob([decryptedData], {
                    type: encryptedFile.type.replace('.enc', ''),
                });

                // Use the decrypted file as needed (e.g., display, download, etc.)
                console.log("Decrypted Blob:", decryptedBlob);
            } catch (error) {
                console.error("Decryption failed:", error);
            }
        };

        reader.onerror = (error) => {
            console.error("File reading error:", error);
        };

        reader.readAsText(encryptedFile);
    }

</script>

<!--    Select File     -->
<!--    Simple form that allows input of file   -->
<h1>Welcome to File Download</h1>
<h2>How to download files using HTML to website?</h2>
<div>
    <button on:click={submit}>Submit</button>
</div>


