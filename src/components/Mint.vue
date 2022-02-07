<template>
  <div>
    <div v-show="account">
      <section>
        <div class="columns is-mobile">
          <div
            class="column is-two-thirds is-three-fifths-tablet is-full-mobile"
          >
            <div class="pt-3 pb-4">
              <div class="columns is-centered mt-5">
                <div class="column is-two-thirds">
                  <div v-if="!isContractChecked">
                    <b-field label="Contract" custom-class="is-large">
                      <b-input v-model="contractAddress"></b-input>
                    </b-field>
                    <b-button class="mt-2" @click="fetchContract()"
                      >Submit</b-button
                    >
                  </div>
                </div>
              </div>

              <div v-if="isContractChecked">
                <div class="columns is-centered">
                  <div class="column is-8">
                    <div class="mb-5">
                      <p class="label" v-if="!fileToMint.name">
                        Choose a file for minting
                      </p>
                      <b-field
                        v-if="!fileToMint.name"
                        class="size-large"
                        custom-class="is-large"
                      >
                        <b-upload v-model="fileToMint" expanded drag-drop>
                          <section class="section">
                            <div class="content has-text-centered">
                              <p>Drop your files here or click to upload</p>
                            </div>
                          </section>
                        </b-upload>
                      </b-field>
                    </div>
                    <div class="mt-5">
                      <b-field
                        label="Name"
                        class="size-large"
                        custom-class="is-large"
                      >
                        <b-input v-model="name"></b-input>
                      </b-field>
                    </div>
                    <div class="mt-5">
                      <b-field label="Description" custom-class="is-large">
                        <b-input
                          maxlength="200"
                          v-model="description"
                          type="textarea"
                        ></b-input>
                      </b-field>
                    </div>
                    <div class="mt-5 mb-5">
                      <b-field label="Amount" custom-class="is-large">
                        <b-input v-model="amount" type="number"></b-input>
                      </b-field>
                    </div>

                    <b-button
                      v-if="ipfsFile && ipfsMetadata && !isMinting"
                      expanded
                      v-on:click="mint"
                      >MINT!</b-button
                    >
                    <b-button
                      v-if="!ipfsFile && !ipfsMetadata && !isUploadingIPFS"
                      expanded
                      v-on:click="uploadFile"
                      >Upload file to IPFS</b-button
                    >
                    <b-button
                      v-if="ipfsFile && !ipfsMetadata && !isUploadingMetadata"
                      expanded
                      v-on:click="createJson"
                      >Upload metadata to IPFS</b-button
                    ><br />

                    <div v-if="fileToMint.name">
                      Selected file: <b>{{ fileToMint.name }}</b>
                    </div>
                    <div v-if="ipfsFile">
                      IPFS hash is:
                      <b
                        ><a
                          :href="'https://ipfs.io/ipfs/' + ipfsFile"
                          target="_blank"
                          >{{ ipfsFile }}</a
                        ></b
                      >
                    </div>
                    <div v-if="isPrepareMinting">
                      Preparing Mint, please wait...
                    </div>
                    <div v-if="isMinting">Minting NFT, please wait...</div>
                    <div v-if="isUploadingIPFS">
                      Uploading file to IPFS, please wait...
                    </div>
                    <div v-if="isUploadingMetadata">
                      Uploading metadata to IPFS, please wait...
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="console-log b-left column p-0">
            <div class="has-text-start">
              <h2 class="mb-2 px-4 pt-5">WHAT'S HAPPENING</h2>
              <div class="mt-5 b-bottom">
                <div class="px-4 mb-5">
                  <h2>Address connected is:</h2>
                  <p style="font-size: 12px">{{ account }}</p>
                  <div class="mt-4" v-if="account && isContractChecked">
                    <p>You are working on contract:</p>
                    <p>
                      <b>{{ contractAddress }}</b>
                    </p>
                    <b-button class="mt-4" @click="changeContract()"
                      >Change Contract</b-button
                    >
                  </div>
                </div>
              </div>
              <div class="px-4 pt-5">
                <div id="printLog"></div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </div>

    <div class="is-flex is-justify-content-center">
      <div class="has-text-centered mt-5" v-if="!account">
        <h3>
          Please connect your Metamask wallet first,<br />window should be open
          automatically or click below button.<br /><br />
        </h3>
        <b-button v-on:click="connect">CONNECT METAMASK</b-button>
      </div>
    </div>
  </div>
</template>

<script>
var Web3 = require("web3");
const ABI_721 = require("../util/abi721.json");
const ABI_1155 = require("../util/abi1155.json");
const axios = require("axios");
const FormData = require("form-data");

export default {
  name: "Mint",
  data() {
    return {
      web3: new Web3(window.ethereum),
      contractAddress: "",
      account: "",
      isContractChecked: "",
      standardContract: "",
      contract: {},
      signature: "",
      code: "",
      fileToMint: {},
      isUploadingIPFS: false,
      isUploadingMetadata: false,
      isMinting: false,
      isPrepareMinting: false,
      name: "",
      ipfsFile: "",
      ipfsMetadata: "",
      description: "",
      axios: axios,
      amount: "",
      infuraURL: "https://ipfs.infura.io:5001/api/v0/add",
    };
  },
  mounted() {
    this.connect();
  },
  methods: {
    printLog(message, value = "") {
      console.log(message, value);
      document.getElementById("printLog").innerHTML =
        message +
        " " +
        value +
        "<div class='console-margin'>" +
        document.getElementById("printLog").innerHTML;
    },

    async connect() {
      const app = this;
      window.ethereum.enable();
      let accounts = await app.web3.eth.getAccounts();
      // this.contract = contract;
      app.account = accounts[0];
      // fare un fetch per vedere se c'è l'indirizzo del contratto nello storage
      let contractChecked = localStorage.getItem("contract");
      let standardContract = localStorage.getItem("standard");
      if (
        contractChecked !== null &&
        contractChecked !== undefined &&
        contractChecked.length > 0
      ) {
        app.isContractChecked = true;
        // console.log(app.isContractChecked);
        app.contractAddress = contractChecked;
        app.standardContract = standardContract;
        app.printLog("Contract address inserted:", standardContract);
      } else {
        app.isContractChecked = false;
      }
    },

    async fetchContract() {
      const app = this;
      app.printLog("fetching contract...");
      try {
        let checkContract1155 = await new app.web3.eth.Contract(
          ABI_1155,
          app.contractAddress,
          {
            gasLimit: "5000000",
          }
        );
        try {
          await checkContract1155.methods.creatorOfToken(0).call();
          app.printLog("starting checking contract 1155");
          localStorage.setItem("contract", app.contractAddress);
          localStorage.setItem("standard", "1155");
          app.printLog("You have inserted a 1155 contract");
          app.isContractChecked = true;
        } catch (e) {
          let checkContract721 = await new app.web3.eth.Contract(
            ABI_721,
            app.contractAddress,
            {
              gasLimit: "5000000",
            }
          );
          try {
            await checkContract721.methods.nftExists("-").call();
            app.printLog("starting checking contract 721");
            localStorage.setItem("contract", app.contractAddress);
            localStorage.setItem("standard", "721");
            console.log("721 is checked", checkContract1155);
            app.isContractChecked = true;
          } catch (e) {
            alert("The contract address that you have inserted it's not valid");
          }
        }
      } catch (e) {
        alert("error");
      }
    },

    async uploadFile() {
      const app = this;
      if (app.fileToMint.name && !app.isUploadingIPFS) {
        //chiedere firma di un messaggio al metamask ("Create metadata for  app.fileToMint.name at new Date().getTime()")
        //per mandare il messaggio fare una chiamata:
        app.code =
          "Create metadata for " +
          app.fileToMint.name +
          " at " +
          new Date().getTime();
        try {
          app.signature = await app.web3.eth.personal.sign(
            app.code,
            app.account
          );
          console.log("signature is:", app.signature);
        } catch (e) {
          alert(e.message);
        }
        if (app.account !== undefined && app.signature.length > 0) {
          app.isUploadingIPFS = true;
          const formData = new FormData();
          formData.append("file", app.fileToMint);
          formData.append("description", app.description);
          formData.append("name", app.name);
          formData.append("message", app.code);
          formData.append("signature", app.signature);
          app.printLog("filename:", app.name);
          app.printLog("description:", app.description);
          app.printLog("Sending request...");
          //come append mettere nome desrizione
          try {
            let response = await axios({
              method: "post",
              url: process.env.VUE_APP_API_URL,
              data: formData,
              headers: {
                "Content-Type": "multipart/form-data",
              },
            });
            app.ipfsFile = response.data.ipfsHash;
            app.isUploadingIPFS = false;
            // file di risposta è già pronto per il minting (app.ipfsMetadata)
            app.ipfsMetadata = true;
          } catch (e) {
            alert("Can't create metadata");
          }
          app.printLog("All data was submitted");
        } else {
          alert("Sign message first!");
        }
      } else {
        alert("Select a file first!");
      }
    },

    async mint() {
      const app = this;
      console.log(app.standardContract);
      if (!app.isMinting) {
        app.isPrepareMinting = true;
        if (app.standardContract === "1155") {
          try {
            app.contract = await new app.web3.eth.Contract(
              ABI_1155,
              app.contractAddress,
              {
                gasLimit: "5000000",
              }
            );
            console.log(app.ipfsMetadata);
            if (app.ipfsFile.length > 0) {
              let prepared = await app.contract.methods
                .prepare(app.account, app.ipfsFile, app.amount)
                .send({ from: this.account });
              app.printLog(
                "Successfully prepared at: " + prepared.transactionHash
              );
              app.isPrepareMinting = false;
              app.isMinting = true;
              let minted = await app.contract.methods
                .mint(app.account, app.ipfsFile, app.amount)
                .send({ from: this.account });
              alert("Successfully minted at: " + minted.transactionHash);
              app.isMinting = false;
            }
          } catch (e) {
            alert(e);
          }
        } else {
          console.log("IM ON 721");
          try {
            app.contract = await new app.web3.eth.Contract(
              ABI_721,
              app.contractAddress,
              {
                gasLimit: "5000000",
              }
            );
            let minted = await app.contract.methods
              .mintNFT(app.account, app.ipfsFile)
              .send({ from: this.account });
            alert("Successfully minted at: " + minted.transactionHash);
            app.isMinting = false;
          } catch (e) {
            alert(e);
          }
        }
      }
    },

    changeContract() {
      localStorage.clear();
      window.location.reload();
    },
  },
};
</script>
