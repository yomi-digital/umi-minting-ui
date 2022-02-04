<template>
  <div>
    <section class="hero">
      <Header />
      <Mint />
    </section>
  </div>
</template>

<script>
// @ is an alias to /src
import Mint from "@/components/Mint.vue";
import Header from "@/components/Header.vue";

export default {
  name: "Home",
  components: {
    Header,
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
