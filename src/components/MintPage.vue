<template>
    <div>
        <button v-if="accounts.length === 0" @click="connectToMetamask()">Connect to Metamask</button>
        <button v-else @click="disconnect()">Disconnect</button>
        <div>Account: {{ accounts[0] }}</div>
        <h1>NFT Minting Page</h1>
        <button @click="mintNft()">Mint NFT</button>
        <hr>

        <table>
            <tr v-for="(nft, index) in nfts" :key="index">
                <td>Token # {{ nft.tokenId }}</td>
                <td><img :src="nft.image" height="150"/></td>
            </tr>
        </table>
        <hr>
        <div v-for="(nft, index) in nfts" :key="index" class="nft-card">
            <h4>Token # {{ nft.tokenId }}</h4>
            <img :src="nft.image" height="150"/>

            <div v-for="(attr, index2) in nft.attributes" :key="index2" class="nft-attr">
                <strong>{{ attr.trait_type }}</strong> - {{ attr.value }}
            </div>
        </div>
    </div>
</template>
<script>
    import { ethers, utils } from 'ethers'
    import erc721Abi from '../contract-abis/erc721.json'

    export default {
        data() {
            return {
                provider: null,
                accounts: [],
                nftContract: null,
                nfts: [],
            }
        },
        methods: {
            async getMintedNfts() {
                var _nfts = []
                var eventFilter = this.nftContract.filters.Minted()
                var logs = await this.nftContract.queryFilter(eventFilter)
                
                for(var i =0; i < logs.length; i++) {
                    var tokenId = logs[i].args.tokenId
                    var tokenUri = await this.nftContract.tokenURI(tokenId)

                    var _response = await fetch(tokenUri)
                    var tokenMetadata = await _response.json()

                    tokenMetadata.tokenId = tokenId
                    _nfts.push(tokenMetadata)
                }
                this.nfts = _nfts
            },
            async mintNft() {
                var signer = this.provider.getSigner()
                var nftContractWithSigner = this.nftContract.connect(signer)

                var transaction = await nftContractWithSigner.mint({
                    value: utils.parseEther('1')
                })
                await transaction.wait()
                window.alert("You have minted a new NFT!")
                this.getMintedNfts()
            },
            disconnect() {
                this.provider = null
                this.accounts = []
                this.nftContract = null
            },
            async connectToMetamask() {
                this.provider = new ethers.providers.Web3Provider(window.ethereum)
                this.accounts = await this.provider.send('eth_requestAccounts', [])

                this.createContractInstance()
            },
            createContractInstance() {
                this.nftContract = new ethers.Contract('0xb25B219DF006185b65c05B900eC527Ee5f7995D9', erc721Abi)
                this.nftContract = this.nftContract.connect(this.provider)

                this.getMintedNfts()
            },
        }
    }
</script>
<style scoped>
    .nft-card {
        background-color: #eee;
        border-radius: 5px;
        box-shadow: #9a9a9a 2px 2px 3px;
        padding: 15px;
        margin-bottom: 15px;
    }

    .nft-attr {
        border: 1px solid black;
        border-radius: 5px;
        margin-bottom: 5px;
    }
</style>