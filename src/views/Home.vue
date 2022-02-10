<template>
  <div>
    <div class="structure">
      <section class="hero">
        <Header />
        <Mint />
      </section>
    </div>
    <Footer />
  </div>
</template>

<script>
// @ is an alias to /src
import Mint from "@/components/Mint.vue";
import Header from "@/components/Header.vue";
import Footer from "@/components/Footer.vue";

export default {
  name: "Home",
  components: {
    Header,
    Mint,
    Footer,
  },
};
</script>

<style scoped>
h1 {
  font-size: 3rem;
  font-weight: 800;
}
</style>
