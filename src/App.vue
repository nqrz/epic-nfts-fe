<script setup>
import { onMounted, ref } from "vue"
import { ethers } from "ethers"
import myEpicNft from "./utils/MyEpicNFT.json"

const currentAccount = ref('')
const allWaves = ref([])

const CONTRACT_ADDRESS = import.meta.env.VITE_CONTRACT_ADDRESS

const checkIfWalletIsConnected = async () => {
  try {
    const { ethereum } = window;

    if (!ethereum) {
      console.log("Make sure you have metamask!");
    } else {
      console.log("We have the ethereum object", ethereum);
    }

    const accounts = await ethereum.request({ method: "eth_accounts"});

    if (accounts.length !== 0) {
      const account = accounts[0];
      console.log("found an authorized account:", account);
      currentAccount.value = account
      setupEventListener();
    } else {
      console.log("No authorized account found")
    }
  } catch (error) {
    console.log(error);
  }
}

const connectWallet = async () => {
  try {
    const { ethereum } = window;

    if (!ethereum) {
      alert("Get MetaMask!");
      return;
    }

    const accounts = await ethereum.request({ method: "eth_requestAccounts" });

    console.log("Connected", accounts[0]);
    currentAccount.value = accounts[0]
    setupEventListener();
  } catch (error) {
    console.log(error)
  }
}

const setupEventListener = async () => {
  try {
    const { ethereum } = window;

    if (ethereum) {
      const provider = new ethers.providers.Web3Provider(ethereum);
      const signer = provider.getSigner();
      const connectedContract = new ethers.Contract(CONTRACT_ADDRESS, myEpicNft.abi, signer);

      connectedContract.on("NewEpicNFTMinted", (from, tokenId) => {
        console.log(from, tokenId.toNumber())
        alert(`Hey there! We've minted your NFT and sent it to your wallet. It may be blank right now. It can take a max of 10 min to show up on OpenSea. Here's the link: https://testnets.opensea.io/assets/${CONTRACT_ADDRESS}/${tokenId.toNumber()}`)
      });

      console.log("Setup event listener!")

    } else {
      console.log("Ethereum object doesn't exist!");
    }
  } catch (error) {
    console.log(error);
  }
}

const handleMint = async () => {
  try {
    const { ethereum } = window;

    if (ethereum) {
      const provider = new ethers.providers.Web3Provider(ethereum);
      const signer = provider.getSigner();
      const connectedContract = new ethers.Contract(CONTRACT_ADDRESS, myEpicNft.abi, signer);

      console.log("Going to pop wallet now to pay gas...")
      let nftTxn = await connectedContract.makeAnEpicNFT();

      console.log("Mining... please wait");
      await nftTxn.wait();

      console.log(`Mined, see transaction: https://rinkeby.etherscan.io/tx/${nftTxn.hash}`);

    } else {
      console.log("Ethereum object doesn't exist!");
    }
  } catch (error) {
    console.log(error);
  }
}

onMounted(() => {
  checkIfWalletIsConnected()
})
</script>

<template>
  <div class="tooltip">
    <a class="link" href="http://buildspace.so" target="_blank" rel="noopener noreferrer">
      <button class="btn">
        🦄 Buildspace
      </button>
    </a>
    <a title="Go to my website" class="link" href="http://nizarbaihaqi.com" target="_blank" rel="noopener noreferrer">
      <button class="btn">
        <img src="./assets/logo-light-16x16.png" alt="Logo Nizar Baihaqi" style="margin-right: 8px;">
        Ijay's Web &#8599;
      </button>
    </a>
  </div>
  <div class="container bg-banner" style="height: 100vh;">
    <h1>This is a random three word NFT in a black box</h1>
    <button v-if="!currentAccount" @click="connectWallet" class="btn">
      Connect Wallet
    </button>
    <button v-else class="btn" @click="handleMint">
      Mint NFT
    </button>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans:ital,wght@0,400;0,700;1,400;1,700&display=swap');

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Noto Sans', sans-serif;
  margin: 0;
  padding: 0;
}

.tooltip {
  position: fixed;
  top: 0;
  right: 0;
  display: flex;
  gap: 10px;
  margin-right: 10px;
}

.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.bg-banner {
  background-image: linear-gradient(to bottom left, whitesmoke, lightblue);
}

.link {
  text-decoration: none;
}

.btn {
  border: none;
  margin-top: 10px;
  padding: 10px;
  padding-top: 8px;
  padding-bottom: 8px;
  border-radius: 4px;
  display: flex;
  align-items: center;
}

.btn:hover {
  cursor: pointer;
}
</style>
