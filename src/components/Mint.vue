<template>
  <div>
    <div v-if="account">
      <b>Welcome back</b><br /><i style="font-size: 12px">{{ account }}</i>
      <div v-if="account && isContractChecked">
        <p>You are working on contract:</p>
        <p>
          <b>{{ contractAddress }}</b>
        </p>
      </div>
      <hr />
      <div v-if="!isContractChecked">
        <b-field label="Contract">
          <b-input v-model="contractAddress"></b-input>
        </b-field>
        <b-button class="mt-2 is-primary" @click="fetchContract()"
          >Submit</b-button
        >
      </div>

      <div v-if="isContractChecked" style="padding: 0 20px">
        <b-field v-if="!fileToMint.name">
          <b-upload v-model="fileToMint" expanded drag-drop>
            <section class="section">
              <div class="content has-text-centered">
                <p>Drop your files here or click to upload</p>
              </div>
            </section>
          </b-upload>
        </b-field>
        <b-field label="Name">
          <b-input v-model="name"></b-input>
        </b-field>
        <b-field label="Description">
          <b-input
            maxlength="200"
            v-model="description"
            type="textarea"
          ></b-input>
        </b-field>
        <b-field label="Amount">
          <b-input v-model="amount" type="number"></b-input>
        </b-field>
        <b-button
          type="is-primary"
          v-if="ipfsFile && ipfsMetadata && !isMinting"
          expanded
          v-on:click="mint"
          >MINT!</b-button
        >
        <b-button
          type="is-primary"
          v-if="!ipfsFile && !ipfsMetadata && !isUploadingIPFS"
          expanded
          v-on:click="uploadFile"
          >Upload file to IPFS</b-button
        >
        <b-button
          type="is-primary"
          v-if="ipfsFile && !ipfsMetadata && !isUploadingMetadata"
          expanded
          v-on:click="createJson"
          >Upload metadata to IPFS</b-button
        ><br />
        <div v-if="fileToMint.name">
          Selected file: <b>{{ fileToMint.name }}</b>
        </div>
        <div v-if="ipfsFile">
          File IPFS hash is:
          <b
            ><a :href="'https://ipfs.io/ipfs/' + ipfsFile" target="_blank">{{
              ipfsFile
            }}</a></b
          >
        </div>
        <div v-if="ipfsMetadata">
          Metadata IPFS hash is:
          <b
            ><a
              :href="'https://ipfs.io/ipfs/' + ipfsMetadata"
              target="_blank"
              >{{ ipfsMetadata }}</a
            ></b
          >
        </div>
        <div v-if="isMinting">Minting NFT, please wait...</div>
        <div v-if="isUploadingIPFS">Uploading file to IPFS, please wait...</div>
        <div v-if="isUploadingMetadata">
          Uploading metadata to IPFS, please wait...
        </div>
      </div>
    </div>
    <div v-if="!account">
      Please connect your Metamask wallet first,<br />window should be open
      automatically or click below button.<br /><br />
      <b-button type="is-primary" v-on:click="connect"
        >CONNECT METAMASK</b-button
      >
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
      // contract: {},
      license: "",
      signature: "",
      code: "",
      licenses: [
        "CC BY",
        "CC BY-SA",
        "CC BY-NC",
        "CC BY-NC-SA",
        "CC BY-ND",
        "CC BY-NC-ND",
        "CC0",
      ],
      fileToMint: {},
      isUploadingIPFS: false,
      isUploadingMetadata: false,
      isMinting: false,
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
    async connect() {
      const app = this;
      window.ethereum.enable();
      let accounts = await app.web3.eth.getAccounts();
      // this.contract = contract;
      app.account = accounts[0];
      // fare un fetch per vedere se c'è l'indirizzo del contratto nello storage
      let contractChecked = localStorage.getItem(
        "contract-1155",
        app.contractAddress
      );
      console.log("Contract address inserted", contractChecked);
      if (
        contractChecked !== null &&
        contractChecked !== undefined &&
        contractChecked.length > 0
      ) {
        app.isContractChecked = true;
        console.log(app.isContractChecked);
      } else {
        app.isContractChecked = false;
      }
    },

    async fetchContract() {
      const app = this;
      console.log("fetching contract");
      try {
        let checkContract1155 = await new app.web3.eth.Contract(
          ABI_1155,
          app.contractAddress,
          {
            gasLimit: "5000000",
          }
        );
        console.log(checkContract1155);
        if (checkContract1155.methods.prepare !== undefined) {
          console.log("starting checking contract 1155");
          localStorage.setItem("contract-1155", app.contractAddress);
          console.log("1155 is checked", checkContract1155);
          app.isContractChecked = true;
        } else {
          let checkContract721 = await new app.web3.eth.Contract(
            ABI_721,
            app.contractAddress,
            {
              gasLimit: "5000000",
            }
          );
          if (checkContract721.methods.returnTokenURI !== undefined) {
            console.log("starting checking contract 721");
            localStorage.setItem("contract-755", app.contractAddress);
            console.log("721 is checked", checkContract1155);
            app.isContractChecked = true;
          } else {
            alert("The contract address that you have inserted it's not valid");
            console.log("KERNEL PANIC");
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
          formData.append("signature", {
            signature: app.signature,
            message: app.code,
          });
          console.log("filename:", app.name);
          console.log("description:", app.description);
          console.log("Im starting to comunicating with API");
          //come append mettere nome desrizione

          axios({
            method: "post",
            url: process.env.VUE_APP_API_URL,
            data: formData,
            headers: {
              "Content-Type": "multipart/form-data",
            },
          }).then(function(response) {
            app.ipfsFile = response.data.Hash;
            app.isUploadingIPFS = false;
            // file di risposta è già pronto per il minting (app.ipfsMetadata)
            app.ipfsMetadata = true;
          });
          console.log("API form submitted");
        } else {
          alert("Sign message first!");
        }
      } else {
        alert("Select a file first!");
      }
    },

    //TO DO al posto di create json fare il mint
    async mint() {
      const app = this;
      if (!app.isMinting) {
        app.isMinting = true;
        // se è 1155
        // prepare (amount, ipfs)
        // poi mint (amount)
        // se è 721
        // mintnft (ipfs)
        try {
          let minted = await app.contract.methods
            .mintOpera(app.ipfsMetadata)
            .send({ from: this.account });
          alert("Successfully minted at: " + minted.transactionHash);
          app.isMinting = false;
        } catch (e) {
          alert(e);
        }
      }
    },
  },
};
</script>
