<script>
    import {Web3Storage} from 'web3.storage'
    import {web3StorageToken} from "../stores/storageStores.js";
    // import crypto from 'crypto';

    let fileVar;

    async function submit() {
        console.log("Submitted");

        if (fileVar) {
            // Encryption
            // let iv = crypto.getRandomValues(new Uint8Array(12));
            // console.log(iv);
            // let key = crypto.subtle.generateKey(
            //     {
            //         name: "AES-GCM",
            //         length: 256, //can be  128, 192, or 256
            //     },
            //     false, //whether the key is extractable (i.e. can be used in exportKey)
            //     ["encrypt", "decrypt"] //can "encrypt", "decrypt", "wrapKey", or "unwrapKey"
            // )
            //     .then(function (key) {
            //         //returns a key object
            //         console.log(key);
            //     })
            //     .catch(function (err) {
            //         console.error(err);
            //     });
            // let encryptedFile = await encryptFile(fileVar, iv, key);

            // File Upload
            await uploadFile(fileVar);
        } else {
            console.log('No file selected')
        }
    }

    // async function encryptFile(file, iv, key) {
    //     const reader = new FileReader();
    //     const fileByteArray = [];
    //     reader.readAsArrayBuffer(file);
    //     reader.onloadend = function (evt) {
    //         if (evt.target.readyState === FileReader.DONE) {
    //             const arrayBuffer = evt.target.result,
    //                 array = new Uint8Array(arrayBuffer);
    //             for (let i = 0; i < array.length; i++) {
    //                 fileByteArray.push(array[i]);
    //             }
    //         }
    //     }
    //     let encryptedFileArray;
    //     crypto.subtle.encrypt(
    //         {
    //             name: "AES-GCM",
    //             iv: iv,
    //             tagLength: 128
    //         },
    //         key, //from generateKey or importKey above
    //         fileByteArray //ArrayBuffer of data you want to encrypt
    //     )
    //         .then(function(encrypted){
    //             //returns an ArrayBuffer containing the encrypted data
    //             console.log(new Uint8Array(encrypted));
    //             encryptedFileArray = encrypted;
    //         })
    //         .catch(function(err){
    //             console.error(err);
    //         });
    //     // convert to file
    //     return new File(encryptedFileArray, file.name, {type: file.type});
    // }

    async function uploadFile(file) {
        const token = web3StorageToken

        if (!token) {
            return console.error('A token is needed. You can create one on https://web3.storage')
        }

        const storage = new Web3Storage({token})
        const cid = await storage.put(file)
        console.log('Content added with CID:', cid)
    }

</script>

<!--    Select File     -->
<!--    Simple form that allows input of file   -->
<h1>Welcome to GFG</h1>
<h2>How to upload files using HTML to website?</h2>
<div>
    <form on:submit={submit}>
        <input
                type="file"
                bind:files={fileVar}/>
        <br/>
        <input type="submit"/>
    </form>
</div>
