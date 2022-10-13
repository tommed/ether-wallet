<template>
  <div class="about">
    <h1>ethers.js test page</h1>
    <p>
      Your balance is: <code>{{ balance }}</code><br/>
      From address: <code>{{ address }}</code><br/>
      From network: <code>{{ network }}</code><br/>
    </p>
    <div class="buttons">
      <button @click="connect">Connect</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { ethers } from 'ethers'

const provider = new ethers.providers.Web3Provider(window.ethereum)
let signer = null // provider.getSigner()
const balance = ref(0)
const address = ref(0)
const network = ref('unknown')
const error = ref(null)

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
</script>