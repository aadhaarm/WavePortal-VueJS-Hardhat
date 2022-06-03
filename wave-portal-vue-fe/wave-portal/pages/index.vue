<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-card>
        <v-card-title class="headline">
          🎵 Wave at me - Dhaa dhinn dhinn dhaaa..!
        </v-card-title>
        <v-card-text>
          <p> I am Aadhaar and I worked on playing Tabla so that's pretty cool right? Connect your Ethereum wallet and
            wave at me!
          </p>
        </v-card-text>
        <v-card-text>
          <ul v-if="currentAccount">
            <li>Total Waves:: {{ waveCount }}</li>
            <li>Collected Wallet:: {{ currentAccount }}</li>
            <li>Wallet Balance:: {{ balance }} ETH</li>
          </ul>
        </v-card-text>

        <form v-if="currentAccount">
          <v-text-field label="Enter your Message" v-model="message" />
          <v-btn className="waveButton" v-on:click=wave color="primary">
            Wave at Me
          </v-btn>
        </form>

        <v-card-actions>
          <v-spacer />
          <v-btn v-if="!currentAccount" v-on:click=connectWallet color="primary" levation="24">
            Connect Wallet
          </v-btn>
        </v-card-actions>
        <v-card-text>
          <ul>
            <li v-for="wave in allWaves" style="padding: 15px;">
              <div>Time: {{ wave.timestamp.toString() }}</div>
              <div>Address: {{ wave.address }}</div>
              <h3>Message: {{ wave.message }}</h3>
            </li>
          </ul>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
import { ethers } from "ethers";
import abi from "~/assets/WavePortal.json";
import Web3 from 'web3';

export default {
  name: 'IndexPage',
  data() {
    return {
      currentAccount: "",
      allWaves: [],
      waveCount: 0,
      message: "",
      contractAddress: "0x291fcD82695Ea2BE42b2f2107BDe355cB093A504",
      contractABI: abi.abi,
      walletAddr: "",
      web3: {},
      balance: 0
    }
  },
  mounted() {
    this.web3 = new Web3(window.ethereum);
    // console.log("Web3", this.web3);
    this.checkIfWalletIsConnected();
    // this.getCount();
    if (window.ethereum) {
      let provider = new ethers.providers.Web3Provider(window.ethereum);
      let signer = provider.getSigner();
      let wavePortalContract = new ethers.Contract(this.contractAddress, this.contractABI, signer);
      wavePortalContract.on("NewWave", this.onNewWave());
    }
  },
  methods: {
    async getAllWaves() {
      try {
        const { ethereum } = window;
        if (ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const wavePortalContract = new ethers.Contract(this.contractAddress, this.contractABI, signer);

          /*
           * Call the getAllWaves method from your Smart Contract
           */
          const waves = await wavePortalContract.getAllWaves();

          /*
           * We only need address, timestamp, and message in our UI so let's
           * pick those out
           */
          let wavesCleaned = [];
          waves.forEach(wave => {
            wavesCleaned.push({
              address: wave.waver,
              timestamp: new Date(wave.timestamp * 1000),
              message: wave.message
            });
          });

          /*
           * Store our data in React State
           */
          wavesCleaned = wavesCleaned.reverse();
          this.setAllWaves(wavesCleaned);
        } else {
          console.log("Ethereum object doesn't exist!")
        }
      } catch (error) {
        console.log(error);
      }
    },
    async checkIfWalletIsConnected() {
      try {
        const { ethereum } = window;

        if (!ethereum) {
          console.log("Make sure you have metamask!");
          return;
        } else {
          console.log("We have the ethereum object", ethereum);
        }

        /*
        * Check if we're authorized to access the user's wallet
        */
        const accounts = await ethereum.request({ method: "eth_accounts" });

        if (accounts.length !== 0) {
          const account = accounts[0];
          console.log("Found an authorized account:", account);
          this.setCurrentAccount(account)
          this.getAllWaves();
          this.getCount();
          this.getBalance();
        } else {
          console.log("No authorized account found")
        }
      } catch (error) {
        console.log(error);
      }
    },
    async connectWallet() {
      try {
        const { ethereum } = window;

        if (!ethereum) {
          alert("Get MetaMask!");
          return;
        }

        const accounts = await ethereum.request({ method: "eth_requestAccounts" });

        console.log("Connected", accounts[0]);
        this.setCurrentAccount(accounts[0]);
        this.getAllWaves();
        this.getCount();
        this.getBalance();
      } catch (error) {
        console.log(error)
      }
    },
    async getCount() {
      try {
        const { ethereum } = window;
        if (ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const wavePortalContract = new ethers.Contract(this.contractAddress, this.contractABI, signer);

          let count = await wavePortalContract.getTotalWaves();
          console.log("getCount() - Retrieved total wave count...", count.toNumber());
          this.setWaveCount(count.toNumber());
        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error);
      }
    },
    async wave() {
      try {
        const { ethereum } = window;

        if (ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const wavePortalContract = new ethers.Contract(this.contractAddress, this.contractABI, signer);

          /*
          * Execute the actual wave from your smart contract
          */
          console.log("Message", this.message);

          const waveTxn = await wavePortalContract.wave(this.message, { gasLimit: 300000 })
          console.log("Mining...", waveTxn.hash);

          await waveTxn.wait();
          console.log("Mined -- ", waveTxn.hash);
          this.getCount();
          this.getAllWaves();

        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error);
      }
    },
    onNewWave(from, timestamp, message) {
      if (from) {
        console.log("NewWave", from, timestamp, message);
      }
      let newWave = {
        address: from,
        timestamp: new Date(timestamp * 1000),
        message: message,
      };
      this.allWaves.unshift(newWave);
      this.getCount();
      this.getBalance();
    },
    setCurrentAccount(acc) {
      this.currentAccount = acc;
    },
    setAllWaves(allWaves) {
      this.allWaves = allWaves;
    },
    setWaveCount(count) {
      this.waveCount = count;
    },
    setMessage(msg) {
      this.message = msg;
    },
    async getBalance() {
      // console.log("currentAccount", this.currentAccount);
      this.web3
        .eth
        .getBalance(this.currentAccount)
        .then((val) => {
          this.balance = this.web3.utils.fromWei(val);
          // console.log(this.balance);
        })
    }
  },
}
</script>
