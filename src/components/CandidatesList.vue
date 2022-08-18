<template>    
    <div>
        <b-button variant="primary" v-if="accounts.length === 0" @click="connectToMetamask()">
            Connect to Metamask            
        </b-button>
        <b-button variant="danger" v-else @click="disconnect()">
            Disconnect from Metamask
        </b-button>
        <!-- <div>
            <b-button v-b-modal.modal-1>Confirmation</b-button>
            <b-modal id="modal-1" title="BootstrapVue">
                <p class="my-4">Hello from modal!</p>
            </b-modal>
        </div> -->
        <hr>
        <div><i>Account: {{ accounts[0] }}</i></div>
        <div><i>Balance: {{ balance }} ETH</i></div>
        <hr>
        <div v-if="accounts.length === 0">
        </div>
        <div v-else class="container">
            <h1>{{ message }}</h1>
            <div class="votecount-text"><i>Number of Candidates: {{ candidatesLength }}</i></div>
            <div><label class="leading">Leading Candidate: {{ winner }}</label></div>
            <div v-if="errorMessage.length > 0">
                Error: {{ errorMessage}}
            </div>
            <div v-if="errorMessage.length > 0">
                <b-modal id="modal-1" title="Error Message">
                    <p class="my-4">{{ errorMessage }}</p>
                </b-modal>
            </div>
            <hr>
            <table class="table table-bordered">
                <tr>
                    <th>#</th>
                    <th>Candidate Photo</th>
                    <th>Candidate Name</th>
                    <th>No. of Votes</th>
                    <th>Status</th>
                    <th></th>
                </tr>
                <tr v-for="(candidate, index) in candidates" :key="index">
                    <td>{{ index + 1 }}</td>
                    <td><img :src="candidate.image" height="150"/></td>
                    <td>{{ candidate.name }}</td>
                    <td>{{ candidate.voteCount }}</td>
                    <td>
                        <label class="leading" v-if="winner === candidate.name">Currently Leading</label>
                    </td>
                    <td>
                        <!-- <b-button class="candidate-button" @click="addVote(index)">Vote {{ candidate.name }}</b-button> -->
                        <b-button class="candidate-button" v-if="errorMessage === ''" @click="addVote(index)">Vote {{ candidate.name }}</b-button>
                        <b-button class="candidate-button" v-else disabled @click="addVote(index)">Vote {{ candidate.name }}</b-button>
                    </td>
                </tr>
            </table>
        </div>
    </div>
</template>

<script>
    import { ethers, utils } from 'ethers'
    import contractAbi from '../contract-abis/ballot.json'

    export default {
        data() {
            return {
                errorMessage: '',
                candidates: [],
                candidatesLength: 0,
                contract: null,
                balance: 0,
                provider: null,
                accounts: [],
                candidate: { 
                    name: 'Admin Is Trator',
                    voteCount: 5
                },
                winner: '',
                message: 'WEB-BASED VOTING APP'
            }
        },
        methods: {
            disconnect() {
                this.accounts = []
                this.balance = 0
                this.provider = null
                this.contract = null
                this.candidates = []

                window.location.reload();
            },
            async getCandidates() {
                this.candidatesLength = await this.contract.getCandidatesLength()
                this.candidates = []
                for(var i = 0; i < this.candidatesLength; i++) {
                    var candidate = await this.contract.candidates(i)
                    var _candidate = {
                        name: candidate.name,
                        voteCount: candidate.voteCount,
                        //image: 'https://avatars.dicebear.com/api/personas/' + candidate.name + '.svg'
                        image: require(`../assets/images/` + candidate.name + `.jpg`),
                    }
                    this.candidates.push(_candidate)
                }
            },
            async getWinner() {
                this.winner = await this.contract.getWinner()
            },
            createContractInstance() {
                this.contract = new ethers.Contract(
                    '0x426E3DaC075d2f19E662D4A862e953212E980Da4', // this address string can be found in the Remix in the Deploy option                    
                    contractAbi // the ABI copied from the Remix, which is pasted in the ballot.json file
                )
                this.contract = this.contract.connect(this.provider)
                this.getCandidates()
                this.getWinner()
            },
            async getBalance() {
                var rawBalance = await this.provider.getBalance(this.accounts[0])
                this.balance = utils.formatEther(rawBalance)
            },
            async connectToMetamask() {
                this.provider = new ethers.providers.Web3Provider(window.ethereum)
                this.accounts = await this.provider.send('eth_requestAccounts', [])

                this.getBalance()
                this.createContractInstance()
            },
            async addVote(candidateIndex) {
                // this.candidate.voteCount = this.voteCountPlusOne()
                var signer = this.provider.getSigner()
                var contractWithSigner = this.contract.connect(signer)
                try {
                    var transaction = await contractWithSigner.vote(candidateIndex)
                    await transaction.wait()
                    this.getCandidates()
                    this.errorMessage = ''
                } catch(error) {
                    this.errorMessage = error.data.message
                }
            },
            voteCountPlusOne() {
                return this.candidate.voteCount + 1
            }
        }
    }
</script>

<style scoped>
    table {
        background-color: #FFFFE0;
    }

    .candidate-button {
        color: black;
        background-color: rgba(57, 57, 230, 0.799);
        padding: 10px 20px;
        margin: 5px;
        border: none;
        border-radius: 5px;       
    }

    /* button:hover {
        background-color: blue;
    } */

    button:active{
        background-color: darkgreen;
    }

    .large-text {
        font-size: 25px;
    }

    .votecount-text {
        font-size: 20px;
    }

    .candidate-card {
        margin: 10px;
        padding: 5px;
        border: 1px solid black;
    }

    .leading {
        font-weight: bold;
        font-style: italic;
        color: red;
    }
</style>