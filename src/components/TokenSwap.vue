<template>
  <div>
    <center>
    <div class="container">
    <h1>Token Swap</h1>
    <br>
    <br>
    <!--Message: {{message}} <br>
    <input v-model="message" />-->

    </div>        
    <button v-if="accounts.length === 0" @click="connectToMetamask()"> Connect to Metamask </button>
    <button v-else @click="disconnect()">Disconnect</button>

    <br>
    <br>
    <br>
    <form action="" @submit.prevent="createSwapOffer()" style="border:solid 1px #ace;width:500px;padding:2%;">
        SwapType: <select v-model="swapForm.swapType">
            <option :value="0">Token A for Token B</option>
            <option :value="1">Token B for Token A</option>
        </select>
        <br>
        <br>
        <div>
            <!--{{swapForm}}-->
        </div>
        amountFrom: <input type="number" v-model="swapForm.amountFrom" />
        <br>
        <br>
        amountTo: <input type="number" v-model="swapForm.amountTo"/>
    <br>
    <br>
        <button type="submit">Create Swap</button>
    </form>



    <div>Account: {{ accounts[0] }} </div>
    <div>TokenA Balance: {{ tokenABalance }} TKA </div>
    <div>TokenB Balance: {{ tokenBBalance }} TKB </div>
    <br>
    <br>
    <br>
    <h4>Swap Offers</h4> 
    <div v-for="(swapOffer,index) in swapOffers" :key="index">
      
      <div v-if="swapOffer.swapType === 0">
         Token A - {{swapOffer.FromTokens}} to B - {{swapOffer.toTokens}}
      </div>
      <div v-else>
         Token B -{{swapOffer.FromTokens}} to A - {{swapOffer.toTokens}}
      </div>
      <button v-if="!swapOffer.isSwapped" @click="fulffillSwap(swapOffer)">fullfillSwap</button>
    </div>


    </center>
  </div>  
</template>
<script>
import TokenAAbi from '../assets/contract-abi/TokenA.json';
import TokenBAbi from '../assets/contract-abi/TokenB.json';
import TokenSwapAbi from '../assets/contract-abi/token-swap.json';

import { ethers,utils } from 'ethers'


export default {
  data() {
    return {
      provider: null,
      accounts: [],
      tokenSwap: null,
      tokenA: null,
      tokenB: null,
      tokenABalance: 0,
      tokenBBalance: 0,
      message: '',
      swapForm: {
        swapType:null,
        amountFrom:0,
        amountTo:0,
      },
      swapOffers: [],
    }
  },
  methods: {
    async fulffillSwap(swapOffer){
      var signer = this.provider.getSigner(signer);
      var tokenSwapWithSigner = this.tokenSwap.connect(signer);

      var tokenTo;
      
      if(swapOffer.swapType === 0){
        tokenTo = this.tokenB;
      }else{
        tokenTo = this.tokenA;
      }

      var tokenToWithSigner = tokenTo.connect(signer);

      var approveTx = await tokenToWithSigner.approve(
        this.tokenSwap.address,swapOffer.amountTo
      )

      await approveTx.wait();

      var transaction = await tokenSwapWithSigner.swap(swapOffer.swapIndex);
      await transaction.wait();
      this.getSwapsList();
      this.getTokenBalances();

    },
    async getSwapsList(){

      var swapLength = await this.tokenSwap.getSwapsLength();
      this.swapOffers = [];
      for(var i = 0 ; i < swapLength; i++){
      var _swap = await this.tokenSwap.swaps(i);

      var swap = {
        swapType: _swap.swapType,
        creator:_swap.creator,
        swappedBy: _swap.swappedBy,
        amountFrom:_swap.amountFrom,
        amountTo:_swap.amountTo,
        isSwapped:_swap.isSwapped,
      };

      swap.swapIndex = i;
      swap.FromTokens = utils.formatUnits(swap.amountFrom,18);
      swap.toTokens = utils.formatUnits(swap.amountTo,18);

      this.swapOffers.push(swap);

      }


    },
    async createSwapOffer(){
        var signer = this.provider.getSigner(signer);
        var tokenSwapWithSigner = this.tokenSwap.connect(signer);

        var amountFrom = utils.parseUnits(String(this.swapForm.amountFrom),18);
        var amountTo = utils.parseUnits(String(this.swapForm.amountTo),18);

        var tokenFrom;
        if(this.swapForm.swapType === 0 ){
            tokenFrom = this.tokenA;
        }else{
            tokenFrom = this.tokenB;
        }

        var tokenFromWithSigner = tokenFrom.connect(signer);
        var approveTx = await tokenFromWithSigner.approve(
            this.tokenSwap.address,
            amountFrom
        )

        await approveTx.wait();

        var transaction = await tokenSwapWithSigner.createSwap(
          this.swapForm.swapType,
          amountFrom,
          amountTo,
        );

        await transaction.wait();
        this.getTokenBalances();
        this.getSwapsList();
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

      this.tokenA = new ethers.Contract('0x4051e8c7E33F806a11fAA93adc7152D43A08D154',TokenAAbi)
      this.tokenA  = this.tokenA.connect(this.provider)

      this.tokenB = new ethers.Contract('0xe2575D17AF2EcefeE959ce2FaCBe03c7f7e7657E',TokenBAbi)
      this.tokenB  = this.tokenB.connect(this.provider)      
        
      this.tokenSwap = new ethers.Contract('0x142441061640652Fa5cdBC39C3C3545A2d215923',TokenSwapAbi )
      this.tokenSwap  = this.tokenSwap.connect(this.provider)

      this.getTokenBalances()
      this.getSwapsList()
    },
    async getTokenBalances() {
        var _tokenABalance = await this.tokenA.balanceOf(this.accounts[0]);
        this.tokenABalance = utils.formatUnits(_tokenABalance,18);

        var _tokenBBalance = await this.tokenB.balanceOf(this.accounts[0]);
        this.tokenBBalance = utils.formatUnits(_tokenBBalance,18);
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

</style>