<template>
  <div>
    <div v-show="account">
      <section>
        <div class="columns is-mobile">
          <div
            class="
              column
              is-two-thirds is-three-fifths-tablet is-full-mobile
              left-column
            "
          >
            <div class="pt-3 pb-4">
              <div class="columns is-centered mt-2">
                <div class="column is-two-thirds">
                  <div v-if="!isContractChecked">
                    <b-field label="Contract" custom-class="is-large">
                      <b-input v-model="contractAddress"></b-input>
                    </b-field>
                    <b-button class="mt-2" @click="fetchContract()"
                      >Select Contract</b-button
                    >
                  </div>
                </div>
              </div>

              <div v-if="isContractChecked">
                <div class="columns is-centered">
                  <div class="column is-9">
                    <div v-if="!ipfsFile && !ipfsMetadata">
                      <div class="mb-5">
                        <p class="label">Choose a file for minting *</p>
                        <div v-if="fileToMint.name">
                          <div
                            class="
                              is-flex
                              is-justify-content-space-between
                              is-align-items-center
                            "
                          >
                            <div v-if="!show" @click="show = true">
                              <b-button class="smaller-btn">show</b-button>
                            </div>
                            <div v-if="show" @click="show = false">
                              <b-button class="smaller-btn">hide</b-button>
                            </div>
                            Selected file: <b>{{ fileToMint.name }}</b>
                            <b-button
                              class="smaller-btn"
                              @click="fileToMint = {}"
                            >
                              X
                            </b-button>
                          </div>
                          <div
                            v-if="preview && show"
                            class="img_container mt-4"
                          >
                            <img :src="preview" />
                          </div>
                        </div>

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
                          label="Name *"
                          class="size-large"
                          custom-class="is-large"
                        >
                          <b-input
                            placeholder="Type a name"
                            v-model="name"
                          ></b-input>
                        </b-field>
                      </div>
                      <div class="mt-5">
                        <b-field label="Description" custom-class="is-large">
                          <b-input
                            maxlength="200"
                            v-model="description"
                            placeholder="Type a description of your nft"
                            type="textarea"
                          ></b-input>
                        </b-field>
                      </div>
                      <div v-if="standardContract === '1155'" class="mt-5 mb-5">
                        <b-field label="Amount *" custom-class="is-large">
                          <b-input
                            v-model="amount"
                            placeholder="Type an amount"
                            type="number"
                          ></b-input>
                        </b-field>
                      </div>
                    </div>

                    <div
                      v-if="ipfsFile"
                      style="
                        padding: 20px;
                        text-align: center !important;
                        font-size: 22px;
                      "
                    >
                      Your metadata are ready!<br />please check it following
                      below link and push "MINT" button whenever you're ready to
                      mint.
                      <hr />
                      <b
                        ><a
                          :href="'https://ipfs.yomi.digital/ipfs/' + ipfsMetadata"
                          target="_blank"
                          >{{ ipfsMetadata }}</a
                        ></b
                      >
                    </div>

                    <b-button
                      v-if="
                        ipfsFile &&
                        ipfsMetadata.length > 0 &&
                        !isMinting &&
                        !isPrepareMinting
                      "
                      expanded
                      v-on:click="mint"
                      >MINT!</b-button
                    >
                    <b-button
                      v-if="!ipfsFile && !ipfsMetadata && !isUploadingIPFS"
                      expanded
                      v-on:click="uploadFile"
                      >Prepare metadata</b-button
                    >
                    <!-- 
                    <div v-if="fileToMint.name">
                      Selected file: <b>{{ fileToMint.name }}</b>
                    </div> -->
                    <div style="text-align: center" v-if="isPrepareMinting">
                      Preparing Mint, please wait...
                    </div>
                    <div style="text-align: center" v-if="isMinting">
                      Minting NFT, please wait...
                    </div>
                    <div style="text-align: center" v-if="isUploadingIPFS">
                      Uploading metadata to IPFS, please wait...
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="column console-log b-left p-0">
            <div class="has-text-start right-column">
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
                <h2 class="mb-2">WHAT'S HAPPENING:</h2>
                <div class="">
                  <p class="pulsing">_</p>
                  <div id="printLog"></div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </div>
    <div
      v-if="!account"
      class="
        is-flex is-justify-content-center is-align-items-center is-full-mobile
      "
      style="height: 80vh"
    >
      <div class="has-text-centered mt-5">
        <h1>
          Please connect your Metamask wallet first,<br />window should be open
          automatically or click below button.<br /><br />
        </h1>
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
      preview: "",
      isUploadingIPFS: false,
      isUploadingMetadata: false,
      isMinting: false,
      isPrepareMinting: false,
      name: "",
      show: false,
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
  watch: {
    fileToMint() {
      try {
        this.preview = URL.createObjectURL(this.fileToMint);
      } catch (e) {
        this.preview = "";
      }
    },
  },
  methods: {
    printLog(message, value = "") {
      console.log(message, value);
      document.getElementById("printLog").innerHTML =
        message +
        "<br> " +
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
      if (
        app.$route.params.contract !== undefined &&
        app.$route.params.contract.indexOf("0x") === 0
      ) {
        localStorage.setItem("contract", app.$route.params.contract);
      }
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
          app.standardContract = "1155";
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
            app.standardContract = "721";
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
      if (
        app.fileToMint.name.length > 0 &&
        app.name.length > 0 &&
        (app.standardContract === "721" ||
          (app.standardContract === "1155" && parseInt(app.amount) > 0)) &&
        !app.isUploadingIPFS
      ) {
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
          app.isUploadingIPFS = false;
          alert(e.message);
        }
        if (app.account !== undefined && app.signature.length > 0) {
          app.isUploadingIPFS = true;
          const formData = new FormData();
          formData.append("file", app.fileToMint);
          formData.append("name", app.name);
          // TODO: this should be implemented in backend too
          formData.append("message", app.code);
          formData.append("signature", app.signature);
          app.printLog("Uploading media to IPFS...");
          //come append mettere nome desrizione
          try {
            const mediaUpload = await axios({
              method: "post",
              url: process.env.VUE_APP_UMI_URL + "/ipfs/upload",
              data: formData,
              headers: {
                "Content-Type": "multipart/form-data",
              },
            });
            app.ipfsFile = mediaUpload.data.ipfs_hash;
            app.printLog("Media uploaded correctly at: " + app.ipfsFile);
            // Uploading whole metadata to IPFS
            const metadataUpload = await axios({
              method: "post",
              url: process.env.VUE_APP_UMI_URL + "/ipfs/nft",
              data: {
                nft: {
                  name: app.name,
                  description: app.description,
                  image: "ipfs://" + app.ipfsFile,
                },
                provider: "pinata",
              },
            });
            console.log(metadataUpload.data);
            app.ipfsMetadata = metadataUpload.data.ipfs_hash;
            app.isUploadingIPFS = false;
            app.printLog("Metadata uploaded correctly at: " + app.ipfsMetadata);
          } catch (e) {
            app.isUploadingIPFS = false;
            alert("Can't create metadata");
          }
          app.printLog("All data was submitted");
        } else {
          alert("Sign message first!");
        }
      } else {
        alert("Fill all required fields!");
      }
    },
    async mint() {
      const app = this;
      console.log(app.standardContract);
      if (!app.isMinting) {
        if (app.standardContract === "1155") {
          app.isPrepareMinting = true;
          try {
            app.contract = await new app.web3.eth.Contract(
              ABI_1155,
              app.contractAddress,
              {
                gasLimit: "5000000",
              }
            );
            if (app.ipfsFile.length > 0) {
              let prepared = await app.contract.methods
                .prepare(app.account, app.ipfsFile, app.amount)
                .send({ from: this.account })
                .on("transactionHash", (tx) => {
                  app.printLog(
                    "Waiting for transaction to be confirmed at: " + tx
                  );
                });
              app.printLog(
                "Successfully prepared at: " + prepared.transactionHash
              );
              app.isPrepareMinting = false;
              app.isMinting = true;
              let minted = await app.contract.methods
                .mint(app.account, app.ipfsFile, app.amount)
                .send({ from: this.account })
                .on("transactionHash", (tx) => {
                  app.printLog(
                    "Waiting for transaction to be confirmed at: " + tx
                  );
                });
              app.printLog("Successfully minted at: " + minted.transactionHash);
              app.isMinting = false;
              app.name = "";
              app.description = "";
              app.amount = 0;
              app.ipfsFile = "";
              app.ipfsMetadata = "";
              app.fileToMint = {};
            }
          } catch (e) {
            app.printLog(e.message);
          }
        } else {
          app.isMinting = true;
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
              .mintNFT(app.account, app.ipfsMetadata)
              .send({ from: this.account })
              .on("transactionHash", (tx) => {
                app.printLog(
                  "Waiting for transaction to be confirmed at: " + tx
                );
              });
            app.printLog("Successfully minted at: " + minted.transactionHash);
            app.isMinting = false;
            app.name = "";
            app.description = "";
            app.amount = 0;
            app.ipfsFile = "";
            app.ipfsMetadata = "";
            app.fileToMint = {};
          } catch (e) {
            app.printLog(e.message);
          }
        }
      }
    },
    changeContract() {
      localStorage.clear();
      location.href = "/";
    },
  },
};
</script>

<style scoped>
#printLog {
  word-break: break-all;
}
</style>
