<template>
  <div class="myHeader p-3">
    <div
      class="ms-5 ps-2 is-flex is-justify-content-flex-start is-align-items-center"
    >
      <img
        src="../img/logo.png"
        alt=""
        style="height: 40px; margin-right: 15px"
      />
    </div>
    <a v-if="!isDashboard" href="/" class="back-button">
      <i class="fas fa-chevron-left"></i>
    </a>
  </div>
</template>

<script>
export default {
  name: "Header",
  data() {
    return {
      isDashboard: "",
    };
  },
  mounted() {
    const app = this;
    console.log(app.$route);
    if (app.$route.name === "Home") {
      app.isDashboard = true;
    }
  },
};
</script>
