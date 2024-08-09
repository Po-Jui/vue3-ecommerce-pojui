<template>
  <v-app class="bg-white">
    <Navbar />
    <v-main>
      <v-container fluid>
        <router-view />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import emitter from "@/methods/emitter";
import Navbar from "@/components/backend/Navbar.vue";
import Toast from "@/alert/Toast";

export default {
  components: {
    Navbar,
    Toast,
  },
  provide() {
    return {
      emitter,
    };
  },
  created() {
    const token = document.cookie.replace(/(?:(?:^|.*;\s*)hexToken\s*=\s*([^;]*).*$)|^.*$/, "$1");
    this.$http.defaults.headers.common.Authorization = token;
    const api = `${process.env.VUE_APP_API}api/user/check`;
    this.$http.post(api).then((res) => {
      if (!res.data.success) {
        this.$router.push("/login");
        Toast.fire({
          icon: "warning",
          title: "您尚未登入，若要進行後台管理請重新登入!",
        });
      }
    });
  },
};
</script>
