<template>
 <div id="main_wrapper">
    <v-container class="white" style="" id="container">
        

    <v-btn
      v-if="accounts.length === 0"
      color="gray"
      class="ma-2 black--text float-right"
      @click="ConnectToMetamask()"
    >
      Connect to Metamask &nbsp;
      <img right dark src="../assets/metamask.png" width="50px" alt="">
    </v-btn>
    <v-btn color="red" class="ma-2 white--text float-right" v-else @click="dialog='true';dialogMsg='Are you sure you want to disconnect?';dialogUse='disconnect'">Disconnect</v-btn>

        <div> 
            <p>Active Account: {{accounts[0]}} - <span style="color:red">{{loggedUser}}</span></p> 
            <!--<p>Wallet Balance: {{balance}}</p>-->            
        </div>
        <v-row no-gutters>
            <br>
             <center><h5>Number of presidential candidates: {{candidatesLength}}</h5></center>
             <br>
        </v-row>
        <v-row no-gutters>
      


        
      <v-col no-gutters v-if="loggedUser=='CHAIRMAN'" cols="12">
      <v-card
       style="padding:2%;"
        outlined
        tile>
        <h3>Chairman Controls</h3>
        <v-switch
          v-model="votingStatus"
          :label="`Enable voting`"
          @change="setVotingStatus(votingStatus)"
        ></v-switch>
        <v-btn @click="getLeading()">Get Leading</v-btn>    
        
      </v-card>
      <br>
      <br>
      
      </v-col>
          <v-col
            v-for="(candidate,index) in candidates" :key="index"
            cols="12"
            sm="4"
          >
            <v-card
              class="pa-2"
              outlined
              tile>
              <span id="indexBadge">Candidate: {{index+1}} </span>
                <br>
                <center>
                    <img :src="candidate.imageUrl" width="100vw" height="100vh" alt="" />
                </center>
                <br>
                Vote count: {{candidate.voteCount}} <br>
                {{candidate.name}}
                <br>

                <!--<v-btn class="float-right" @click="addVote(index)">Vote</v-btn>-->
                
                <v-btn class="float-right" @click="dialog='true';dialogMsg='Are you sure you want to vote '+candidate.name+'?';selectedCandidate=index;dialogUse='vote'">Vote</v-btn>
                <br>
                <br>
            </v-card>
          </v-col>
        </v-row>
     </v-container>

    <v-dialog
      v-model="dialog"
      width="500"
    >

      <v-card>
        <v-card-text>
            <br>
            {{dialogMsg}}
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            v-if="dialogUse=='disconnect'"
            color="primary"
            text
            @click="disconnect()"
          >            Yes
          </v-btn>
          <v-btn
            v-else-if="dialogUse=='vote'"
            color="primary"
            text
            @click="addVote(selectedCandidate)"
          >
            Yes
          </v-btn>


          <v-btn
            v-if="dialogUse=='Winner'"
            color="primary"
            text
            @click="dialog = false" 
          >
            Close
          </v-btn>
          <v-btn
            v-else
            color="primary"
            text
            @click="dialog = false" 
          >
            No
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>


    <v-dialog
      v-model="loaderDialog"
      hide-overlay
      persistent
      width="300"
    >
      <v-card
        color="primary"
        dark
      >
        <v-card-text>
          Please stand by
          <v-progress-linear
            indeterminate
            color="white"
            class="mb-0"
          ></v-progress-linear>
        </v-card-text>
      </v-card>
    </v-dialog>


 </div>

</template>

<script>
import {ethers,utils} from 'ethers';
import contractABI from '../assets/contract-abi/ballot.json';


export default {
name: 'CandidatesList',
  data() {
    return {
        errorMessage:[],
        message: 'hello world',
        numVariable: 10,
        //candidate: {name:"Pacman",voteCount:10},
        provider: null,
        accounts: [],
        balance:0,
        candidatesLength:0,
        candidates: [],
        dialog:false,
        loaderDialog:false,
        dialogMsg:'',
        selectedCandidate: [],
        dialogUse:'',
        loggedUser: '',
        votingStatus:false,
    }
  },
  methods:{
    async getCandidates(){
        this.candidatesLength = await this.contract.getCandidatesLength();
        this.candidates = [];



        for(var i = 0; i<this.candidatesLength;i++){
            var candidate = await this.contract.candidates(i);
            //console.log(candidate);

            var _candidate = {
                name: candidate.name,
                voteCount: candidate.voteCount,
                imageUrl: candidate.imageUrl,
            }   

            this.candidates.push(_candidate);
        }

        
        
    },
    CreateContractInstance(){
        var contractAddress = '0x1F6eC70596D41b6d7b78B06e6aA0f8DCa24578BA';
        
        this.contract = new ethers.Contract(
        contractAddress,
        contractABI
        );

        this.contract = this.contract.connect(this.provider);
        this.getCandidates();
    },
    async addVote(candidateIndex){

        var signer = this.provider.getSigner();
        var contractWithSigner = this.contract.connect(signer);
        try{
            var transaction = await contractWithSigner.vote(candidateIndex);
            await transaction.wait();
            this.getCandidates();
            this.dialog = false;
        }catch(error){
            //console.log(error);
            //return this.errorMessage = error.data.message;
            if(this.votingStatus == false){
               alert("Voting status is off!");
            }else{
              alert('you already voted!');
            }
           
        }


    },
    voteCountPlusOne(){
        return this.candidate.voteCount+1;
    },
    deduct(){
        return this.candidate.voteCount = this.candidate.voteCount-1;
    },
    reset(){
        return this.candidate.voteCount = 0 ;
    },
    async ConnectToMetamask(){

      this.provider = new ethers.providers.Web3Provider(window.ethereum);
      this.accounts = await this.provider.send('eth_requestAccounts',[]);
      //this.balance = await this.getBalance();
      if(this.CreateContractInstance()){
          this.candidatesLength = this.CreateContractInstance();
      }

      this.IdentifyUser();
      
        
    },
    async getBalance(){
        var RawBalance = await this.provider.getBalance(this.accounts[0]);
        return utils.formatEther(RawBalance);
    },
    disconnect(){
        //if(confirm('Are you sure you want to disconnect session?')){
        this.balance = 0;
        this.contract = null;
        this.provider = null;
        this.candidates = [];
        this.accounts = [];
        this.candidatesLength = 0;
        this.dialog = false;
        //}

    },
    async IdentifyUser(){

      var signer = this.provider.getSigner();
      var contractWithSigner = this.contract.connect(signer);
      var chairperson = await contractWithSigner.chairperson();

      if(this.accounts[0].toLowerCase() === chairperson.toLowerCase()){
          this.loggedUser = 'CHAIRMAN';
      }else{
           this.loggedUser = 'CLIENT';
      }

    },
    async setVotingStatus(votingStatus){
      var signer = this.provider.getSigner();
      var contractWithSigner = this.contract.connect(signer);
      await contractWithSigner.setVotingState(votingStatus);
      if(votingStatus == true){
        this.votingStatus = true;
      }else{
        this.votingStatus = false;
      }
      //console.log(signer);
      //alert(votingStatus)

    },
    async getLeading(){

      var signer = this.provider.getSigner();
      var contractWithSigner = this.contract.connect(signer);
      var leading = await contractWithSigner.getWinner();

      this.dialogUse = "Winner";
      this.dialogMsg = "The leading is "+ leading;
      this.dialog = true;

    }
  }
}
</script>
<style scoped>
#main_wrapper{
    padding-top: 5px;
    padding-bottom: 5px;
    position:absolute;
    z-index: 2;
    top:0px;
    left:0px;
    width: 100%;
    height: 100vh;
    background:rgba(0,0,0,0.5);
    overflow-y: scroll;
}
#indexBadge{
background:#00326b;
color:#fff;
font-size:10px;
padding: 2%;

}

</style>