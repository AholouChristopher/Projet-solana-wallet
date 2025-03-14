<script setup>

import {ref, onMounted } from "vue";
import { Connection, clusterApiUrl, PublicKey, LAMPORTS_PER_SOL, Transaction, SystemProgram } from "@solana/web3.js";

import { Buffer } from "buffer";
window.Buffer = Buffer;

const walletAddress = ref(null);
const balance = ref(0);
const recipient = ref(null);
const amount = ref(0);
const connection = new Connection(clusterApiUrl("devnet"),"confirmed");

//Connexion au wallet Phantom
const connectWallet = async () => {
  if (window.solana && window.solana.isPhantom) {
    try {
      const response = await window.solana.connect();
      walletAddress.value = response.publicKey.toString();
      getBalance()
    } catch (error) {
      console.error("Erreur connection Wallet :", error);
    }
  } else {
    alert("Installe Phantom Wallet !");
  }
}

// Fonction getBalance = récupère le solde du wallet
const getBalance = async () => {
  if (walletAddress.value) {
    const publicKey =  walletAddress.value;
    const solBalance = await connection.getBalance(new PublicKey(publicKey)); // Récupère le solde du wallet
    balance.value = solBalance / LAMPORTS_PER_SOL; // Utilise LAMPORTS_PER_SOL au lieu de 10**9
  }
}

//Fonction envoi Des SOL

const sendSol = async () => {

  if(!recipient.value || !amount.value) {
    alert("Veuillez remplir tous les champs");
    return;
  }
  const to = new PublicKey(recipient.value); // Adresse du destinataire
  const lamports = parseFloat(amount.value) * LAMPORTS_PER_SOL;

  try {
    console.log("Début try");

    const transaction =  new Transaction().add(
      SystemProgram.transfer({
        fromPubkey: new PublicKey(walletAddress.value),
        toPubkey: to,
        lamports: lamports
      })
    );
    console.log("Transaction créée");

    transaction.feePayer = new PublicKey(walletAddress.value); 
    const { blockhash } = await connection.getLatestBlockhash();
    transaction.recentBlockhash = blockhash;

    const signedTransaction = await window.solana.signTransaction(transaction);
    const signature = await connection.sendRawTransaction(signedTransaction.serialize());
    alert(`Transaction envoyée ! Suivi : https://explorer.solana.com/tx/${signature}?cluster=devnet`);
    getBalance(); // Met à jour le solde
    console.log(`https://explorer.solana.com/tx/${signature}?cluster=devnet`);
  } catch (error) {
    console.error("Erreur de transaction:", error);
    alert("Erreur lors de l'envoi des SOL");
  }
};

// verification de la connexion au wallet Phantom
onMounted(() => {
  if (window.solana && window.solana.isPhantom) {
    window.solana.connect({ onlyIfTrusted: true })
      .then((response) => {
        walletAddress.value = response.publicKey.toString();
        getBalance();
      })
      .catch((error) => {
        console.error("Erreur lors de la connexion automatique :", error);
      });
  }})
</script>

<template>
  <div class="app">
    <h1>Mini Wallet Solana</h1>
    <button v-if="!walletAddress" @click="connectWallet">Connecter Phantom Wallet</button>
    <div v-else>
      <p><strong>Adresse :</strong> {{ walletAddress }}</p>
      <p><strong>Solde :</strong> {{ balance }} SOL</p>

      <h2>Envoyer des SOL</h2>
      <input v-model="recipient" placeholder="Adresse du destinataire" />
      <input v-model="amount" type="number" placeholder="Montant (SOL)" />
      <button @click="sendSol">Envoyer</button>
    </div>
  </div>
</template>


<style scoped>
  .app {
    text-align: center;
    margin-top: 50px;
  }
  input {
    display: block;
    margin: 10px auto;
    padding: 8px;
    width: 80%;
  }
  button {
    background-color: #512da8;
    color: white;
    padding: 10px;
    border: none;
    cursor: pointer;
  }
</style>
