<script>
    import {Web3Storage} from 'web3.storage'
    import {web3StorageToken} from "../stores/storageStores.js";
    import CryptoJS from 'crypto-js';

    let files;
    let encryptedFile;
    let decryptedBlob;

    async function submit() {
        await downloadFiles();
        await decryptFile();

        await waitForElement();
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
        const cid = "bafybeibsjzmj56folulqfyatyrf5mr7bp3hgnu62z2psh36ptt4fqvye7i";
        const res = await storage.get(cid)
        const files = await res.files()
        encryptedFile = files[0];
        for (const file of files) {
            console.log(`${file.cid} -- ${file.path} -- ${file.size}`)
            // donwload file
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
        const password = "YourEncryptionPassword"; // Replace with your encryption password

        if (!encryptedFile) {
            console.error("No encrypted file available.");
            return;
        }

        const reader = new FileReader();
        reader.onload = () => {
            try {
                const decrypted = CryptoJS.AES.decrypt(reader.result, password);
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


