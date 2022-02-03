<template>
  <div class="home container">
    <!-- <img style="width:150px" src="../assets/logo.jpg"> -->
    <h1 class="mb-5">UMi | Minting NFTs UI</h1>
    <Mint />
  </div>
</template>

<script>
// @ is an alias to /src
import Mint from "@/components/Mint.vue";

export default {
  name: "Home",
  components: {
    Mint,
  },
};
</script>

<style scoped>
h1 {
  font-size: 3rem;
  font-weight: 800;
}
</style>
