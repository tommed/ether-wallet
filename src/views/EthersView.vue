<template>
  <div class="about">
    <h1>ethers.js test page</h1>
    <div v-if="error">ERROR: {{ error }}</div>
    <p>
      Your balance is: <code>{{ balance }}</code><br/>
      From address: <code>{{ address }}</code><br/>
      From network: <code>{{ network }}</code><br/>
      Signing <code>{{ messageToSign }}</code> as <code>{{ signedMessage }}</code><br/>
      Message from Contract: <code>{{ messageFromContract }}</code><br/>
    </p>

    <p>
      NOTE: before you click connect, please open your wallet and switch to the
      goerli network. This is where the API is deployed to.
    </p>

    <div class="buttons">
      <button @click="connect">Connect</button>
      <button @click="sign">Sign</button>
      <button @click="callContract">Call Contract</button>
    </div>
  </div>
</template>

<script setup>
// we need this to make mutable variables in VueJS
import { ref } from 'vue'

// we need this to access the ethereum network
import { ethers } from 'ethers'

// when building our smart contract, it produced an aritfact .json for the contract
// this contains what is known as an 'ABI' - which is effectively the signature/interface
// definition for the contract. We need to import this here to show ether.js which
// methods (and their arguments) are available from our contract and how to call them.
// this is done over a JSONRPC API
import CLHelloWorld from '@/contracts/CLHelloWorld.json'

// example:
// { address: 'YOUR_SMART_CONTRACT_ADDRESS' }
// I created this JSON file to keep the latest contract address
// so it can be imported into this page. The smart contract address
// is like a pointer to the API itself on the blockchain.
import CLHelloWorldAddr from '@/contracts/CLHelloWorldAddr.json'

// I deployed my smart contract to the GOERLI network, so will put it in here.
// This isn't enforced by MetaMask (no idea why?) So i need to be sure the
// wallet is connected to this network before I click the connect button above.
// `window.ethereum` is the js hook which MetaMask (and other wallets) add
// to each page so you can call it from a dapp.
const provider = new ethers.providers.Web3Provider(window.ethereum, 'goerli')

// vuejs state variables
let signer = null
const balance = ref(0)
const address = ref(0)
const network = ref('unknown')
const error = ref(null)
const messageToSign = "hello world"
const signedMessage = ref(null)
const messageFromContract = ref(null)

// connect routine
async function connect() {
  try {
    await provider.send("eth_requestAccounts", [])
    signer = provider.getSigner()
    network.value = (await provider.getNetwork()).name
    balance.value = await signer.getBalance()
    address.value = await signer.getAddress()
  } catch(e) {
    error.value = e.message
  }
}

// singing a message using ether.js
// this is useful to prove the address of a user rather than just
// taking their word they are the address they claim to be.
// Using this signature isn't completed yet, to be done!
async function sign () {
  // sign a hash?
  //const messageHash = ethers.utils.id(messageToSign)
  //const messageHashBytes = ethers.utils.arrayify(messageHash)
  // sign actual string (so users can see what they're signing in MetaMask)
  const messageHashBytes = ethers.utils.toUtf8Bytes(messageToSign)
  signedMessage.value = await signer.signMessage(messageHashBytes)
}

// Shows how to call my smart contract and retrieve the current value of the
// `message` state variable.
async function callContract () {
  const addr = CLHelloWorldAddr.address
  const contract = new ethers.Contract(addr, CLHelloWorld.abi, signer)
  messageFromContract.value = await contract.message();
}
</script>