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
    <div class="buttons">
      <button @click="connect">Connect</button>
      <button @click="sign">Sign</button>
      <button @click="callContract">Call Contract</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { ethers } from 'ethers'
import CLHelloWorld from '@/contracts/CLHelloWorld.json'
import CLHelloWorldAddr from '@/contracts/CLHelloWorldAddr.json'

const provider = new ethers.providers.Web3Provider(window.ethereum, 'goerli')
let signer = null
const balance = ref(0)
const address = ref(0)
const network = ref('unknown')
const error = ref(null)
const messageToSign = "hello world"
const signedMessage = ref(null)
const messageFromContract = ref(null)

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

async function sign () {
  // sign a hash?
  //const messageHash = ethers.utils.id(messageToSign)
  //const messageHashBytes = ethers.utils.arrayify(messageHash)
  // sign actual string (so users can see what they're signing in MetaMask)
  const messageHashBytes = ethers.utils.toUtf8Bytes(messageToSign)
  signedMessage.value = await signer.signMessage(messageHashBytes)
}

async function callContract () {
  const addr = CLHelloWorldAddr.address
  const contract = new ethers.Contract(addr, CLHelloWorld.abi, signer)
  messageFromContract.value = await contract.message();
}
</script>