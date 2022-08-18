<template>
    <div>
        <button v-if="accounts.length === 0" @click="connectToMetamask()"> Connect to Metamask </button>
        <button v-else @click="disconnect()">Disconnect</button>
        <div>Account: {{ accounts[0] }} </div>
        <div>TokenA Balance: {{ tokenABalance }} TKA </div>
        <div>TokenB Balance: {{ tokenBBalance }} TKB </div>

        <div class="container">
            <h1>Token Swap</h1>

            <form @submit.prevent="createSwapOffer()">
                <div>{{ swapForm }}</div>
                <hr>
                <div>Swap Type</div>
                <select v-model="swapForm.swapType">
                    <option :value="0">Token A for Token B</option>
                    <option :value="1">Token B for Token A</option>
                </select>
                <br><br>
                <div>Amount From</div>
                <input v-model.number="swapForm.amountFrom"/>
                <br><br>
                <div>Amount To</div>
                <input v-model.number="swapForm.amountTo"/>
                <br>
                <div>
                    <button type="submit">Create Swap</button>
                </div>
            </form>

            <div v-for="(swapOffer, index) in swapOffers" :key="index" class="swap-card">
                <div>Swap Offer By: {{ swapOffer.creator }}</div>
                <div v-if="swapOffer.swapType === 0">
                    {{ swapOffer.fromTokens }} Token A for {{ swapOffer.toTokens }} Token B
                </div>
                <div v-else-if="swapOffer.swapType === 1">
                    {{ swapOffer.fromTokens }} Token B for {{swapOffer.toTokens }} Token A
                </div>

                <button v-if="!swapOffer.isSwapped" @click="fulfillSwap(swapOffer)">
                    Swap
                </button>
            </div>
        </div>
    </div>  
</template>

<script>
    import { ethers, utils } from 'ethers'
    import tokenAAbi from '../contract-abis/tokenA.json'
    import tokenBAbi from '../contract-abis/tokenB.json'
    import tokenSwapAbi from '../contract-abis/token-swap.json'

    export default {
        data() {
            return {
                swapForm: {
                    swapType: null,
                    amountFrom: 0,
                    amountTo: 0,
                },
                swapOffers: [],
                message: '',
                provider: null,
                accounts: [],
                tokenSwap: null,
                tokenA: null,
                tokenB: null,
                tokenABalance: 0,
                tokenBBalance: 0,
            }
        },
        methods: {
            async fulfillSwap(swapOffer) {
                var signer = this.provider.getSigner()
                var tokenSwapWithSigner = this.tokenSwap.connect(signer)

                var tokenTo 
                if(swapOffer.swapType === 0) {
                    tokenTo = this.tokenB
                } else if(swapOffer.swapType === 1) {
                    tokenTo = this.tokenA
                }
                var tokenToWithSigner = tokenTo.connect(signer)
                var approveTx = await tokenToWithSigner.approve(this.tokenSwap.address, swapOffer.amountTo)
                await approveTx.wait()

                var transaction = await tokenSwapWithSigner.swap(swapOffer.swapIndex)
                await transaction.wait()
                this.getSwapsList()
                this.getTokenBalances()
            },
            async getSwapsList() {
                var swapsLength = await this.tokenSwap.getSwapsLength()
                this.swapOffers = []
                for(var i = 0; i < swapsLength; i++) {
                    var _swap = await this.tokenSwap.swaps(i)
                    var swap = {
                        swapType: _swap.swapType,
                        creator: _swap.creator,
                        swappedBy: _swap.swappedBy,
                        amountFrom: _swap.amountFrom,
                        amountTo: _swap.amountTo,
                        isSwapped: _swap.isSwapped,
                    }
                    swap.swapIndex = i
                    swap.fromTokens = utils.formatUnits(swap.amountFrom, 18)
                    swap.toTokens = utils.formatUnits(swap.amountTo, 18)
                    this.swapOffers.push(swap)
                }
            },
            async createSwapOffer() {
                var signer = this.provider.getSigner(signer)
                var tokenSwapWithSigner = this.tokenSwap.connect(signer)
                
                var amountFrom = utils.parseUnits(String(this.swapForm.amountFrom), 18)
                var amountTo = utils.parseUnits(String(this.swapForm.amountTo), 18)

                var tokenFrom
                if(this.swapForm.swapType === 0) {
                    tokenFrom = this.tokenA
                } else if(this.swapForm.swapType === 1) {
                    tokenFrom = this.tokenB
                }

                var tokenFromWithSigner = tokenFrom.connect(signer)
                var approveTx = await tokenFromWithSigner.approve(
                    this.tokenSwap.address,
                    amountFrom
                )
                await approveTx.wait()

                var transaction = await tokenSwapWithSigner.createSwap(
                    this.swapForm.swapType,
                    amountFrom,
                    amountTo,
                )
                await transaction.wait()
                this.getTokenBalances()
                this.getSwapsList()
            },
            async connectToMetamask() {
                this.provider = new ethers.providers.Web3Provider(window.ethereum)
                this.accounts = await this.provider.send('eth_requestAccounts', [])

                this.createContractInstances()
            },
            disconnect() {
                this.provider = null
                this.accounts = []

                this.tokenSwap = null
                this.tokenA = null
                this.tokenB = null

                this.tokenABalance = 0
                this.tokenBBalance = 0
            },
            createContractInstances() {
                this.tokenSwap = new ethers.Contract('0x3f1A0Dda9eAbf79725d4A885A1DFe1dc91038420', tokenSwapAbi)
                this.tokenSwap  = this.tokenSwap.connect(this.provider)

                this.tokenA = new ethers.Contract('0xb47fd2bddF55B2819D5c6bD2D2742b5e55b528B5', tokenAAbi)
                this.tokenA  = this.tokenA.connect(this.provider)

                this.tokenB = new ethers.Contract('0xf665b535F0e31cB02aa066BECc15f2A1B5A345be', tokenBAbi)
                this.tokenB  = this.tokenB.connect(this.provider)

                this.getTokenBalances()
                this.getSwapsList()
            },
            async getTokenBalances() {
                var _tokenABalance = await this.tokenA.balanceOf(this.accounts[0])
                this.tokenABalance = utils.formatUnits(_tokenABalance, 18)

                var _tokenBBalance = await this.tokenB.balanceOf(this.accounts[0])
                this.tokenBBalance = utils.formatUnits(_tokenBBalance, 18)
            },
        },
    }
</script>

<style scoped>
    button {
        color: white;
        background-color: green;
        padding: 10px 20px;
        margin: 5px;
        border: none;
        border-radius:  5px;
    }

    button:active {
        background-color: darkgreen;
    }

    .swap-card {
        margin-bottom: 10px;
        border: 1px solid black;
    }
</style>