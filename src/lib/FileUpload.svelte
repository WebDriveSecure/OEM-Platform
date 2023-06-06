<script>
    import {getFilesFromPath, Web3Storage} from 'web3.storage'
    import {web3StorageToken} from "/~/stores/storageStores.js";

    async function uploadFile(filePath) {
        const token = web3StorageToken

        if (!token) {
            return console.error('A token is needed. You can create one on https://web3.storage')
        }

        const storage = new Web3Storage({token})
        // Must be stored as an array in order to push
        const files = []

        const pathFiles = await getFilesFromPath(filePath)
        files.push(...pathFiles)

        console.log(`Uploading ${files.length} files`)
        const cid = await storage.put(files)
        console.log('Content added with CID:', cid)
    }
</script>

<!--    Select File     -->
