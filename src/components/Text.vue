<template>
  <h1>Buy me a coffee</h1>
  <div v-if="currentAccount == ''">
    <input
      class="button-primary"
      type="button"
      value="Connect Wallet"
      style="margin-bottom: 2em;"
      v-on:click="connectWallet"
    />
    <p>Hi! I'm Juan! I'm a full-stack developer and I love coffee.</p>
    <p>I also love new technologies (like Blockchain and DeFi) and I'm always looking for new projects.</p>
    <p>Have a great day!</p>
  </div>

  <div v-else>
    <h5>Only Polygon/Matic is accepted!</h5>
    <div class="row">
      <div class="one-third column">&nbsp;</div>
      <div class="one-third column">
        <label for="Name">Your Name</label>
        <input
          class="u-full-width"
          type="email"
          placeholder="Your beautiful name goes here"
          id="Name"
        />
      </div>
      <div class="one-third column">&nbsp;</div>
    </div>
    <div class="row">
      <div class="one-third column">&nbsp;</div>
      <div class="one-third column">
        <label for="Message">Message</label>
        <textarea class="u-full-width" placeholder="Enjoy your coffee!" id="Message"></textarea>
      </div>
      <div class="one-third column">&nbsp;</div>
    </div>
    <input
      class="button-primary"
      type="button"
      value="Buy a coffee"
      style="margin-bottom: 2em;"
      v-on:click="buyCoffee"
    />
  </div>

  <div v-for="coffee,index in allCoffee" v-bind:key="index">
    <div class="row">
      <div class="columns three">&nbsp;</div>
      <div class="columns six timeLineBox">
        <p>Supporter: {{ coffee.name }}</p>
        <p>Message: {{ coffee.message }}</p>
        <p>Address: {{ coffee.address }}</p>
        <p>TimeStamp: {{ coffee.timestamp.toString() }}</p>
      </div>
      <div class="columns three">&nbsp;</div>
    </div>
  </div>
  <p>
    <a href="https://vitejs.dev/guide/features.html" target="_blank">My Github</a>
    |
    <a href="https://v3.vuejs.org/" target="_blank">My LinkedIn</a>
  </p>
</template>

<style scoped>
a {
  color: #7a59f1;
}
.timeLineBox {
  border-radius: 25px;
  padding: 2em;
  margin-bottom: 3em;
  margin-top: 3em;
  box-shadow: 10px 10px 47px -10px rgba(77, 57, 72, 0.2);
}
</style>

<script>
import abi from '../utils/CoffeeMachine.json'
import { createToast as toast } from 'mosha-vue-toastify';
import 'mosha-vue-toastify/dist/style.css'
import { ethers } from "ethers";

export default {
  name: 'Text',
  data() {
    return {
      name: "",
      message: "",
      contractABI: abi.abi,
      contractAddress: '0xE43C2D17a228251b22f1BAcf8db89B6cEE2274C9',
      currentAccount: '',
      allCoffee: [],
    }
  },
  methods: {
    async checkIfWalletIsConnected() {
      try {
        const { ethereum } = window;

        /*
         * Check if we're authorized to access the user's wallet
         */
        const accounts = await ethereum.request({ method: "eth_accounts" });

        if (accounts.length !== 0) {
          const account = accounts[0];
          this.currentAccount = (account);
          toast("🦄 Wallet is Connected");
        }
      } catch (error) {
        toast(`${error.message}`, { type: "danger" });
      }
    },
    async connectWallet() {
      try {
        const { ethereum } = window;

        if (!ethereum) {
          toast("Make sure you have MetaMask Connected");
          return;
        }

        const accounts = await ethereum.request({
          method: "eth_requestAccounts",
        });
        this.currentAccount = (accounts[0]);
        toast("🦄 Wallet is Connected");
      } catch (error) {
        console.log(error);
      }
    },
    async buyCoffee() {
      try {
        const { ethereum } = window;

        if (ethereum) {
          await this.detectChain(ethereum);
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const coffeeMachineContract = new ethers.Contract(
            this.contractAddress,
            this.contractABI,
            signer
          );

          let count = await coffeeMachineContract.getTotalCoffee();
          console.log("Retrieved total coffee count...", count.toNumber());

          /*
           * Execute the actual coffee from your smart contract
           */
          const coffeeTxn = await coffeeMachineContract.buyCoffee(
            this.message !== "" ? this.message : "Enjoy Your Coffee",
            this.name !== "" ? this.name : "Anonymous",
            ethers.utils.parseEther("0.001"),
            {
              gasLimit: 300000,
            }
          );
          console.log("Mining...", coffeeTxn.hash);

          toast("Sending Fund for coffee...");
          await coffeeTxn.wait();

          console.log("Mined -- ", coffeeTxn.hash);

          count = await coffeeMachineContract.getTotalCoffee();

          console.log("Retrieved total coffee count...", count.toNumber());

          this.message = "";
          this.name = "";

          toast("Coffee Purchased!", {
            type: "success",
          });
        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        toast(`${error.message}`, {
          type: "danger",
        });
      }
    },
    async getAllCoffee() {
      try {
        const { ethereum } = window;
        if (ethereum) {
          await this.detectChain(ethereum);
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const coffeeMachineContract = new ethers.Contract(
            this.contractAddress,
            this.contractABI,
            signer
          );

          /*
           * Call the getAllCoffee method from your Smart Contract
           */
          const coffees = await coffeeMachineContract.getAllCoffee();

          /*
           * We only need address, timestamp, name, and message in our UI so let's
           * pick those out
           */
          const coffeeCleaned = coffees.map((coffee) => {
            return {
              address: coffee.giver,
              timestamp: new Date(coffee.timestamp * 1000),
              message: coffee.message,
              name: coffee.name,
            };
          });

          /*
           * Store our data in React State
           */
          this.allCoffee = coffeeCleaned;
        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error);
      }
    },
    async detectChain(ethereum) {
      const chainId = await ethereum.request({ method: 'eth_chainId' });
      if (!["0x13881", "0x89"].includes(chainId)) {
        throw new Error("Only Polygon/Matic is accepted");
      }
    }
  },
  mounted() {
    let coffeeMachineContract;
    this.getAllCoffee();
    this.checkIfWalletIsConnected();

    const onNewCoffee = (from, timestamp, message, name) => {
      console.log("NewCoffee", from, timestamp, message, name);
      this.allCoffee = [
        ...this.allCoffee,
        {
          address: from,
          timestamp: new Date(timestamp * 1000),
          message: message,
          name: name,
        },
      ];
    };

    if (window.ethereum) {
      const provider = new ethers.providers.Web3Provider(window.ethereum);
      const signer = provider.getSigner();

      coffeeMachineContract = new ethers.Contract(
        this.contractAddress,
        this.contractABI,
        signer
      );

      coffeeMachineContract.on("NewCoffee", onNewCoffee);
    }


  },
}
</script>