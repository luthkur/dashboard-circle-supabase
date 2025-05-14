<template>
  <div class="container" style="padding: 50px 0 100px 0">
    <Profile v-if="store?.user?.id" />
    <Auth v-else />
  </div>
</template>

<script>
import { store } from "./store";
import { supabase } from "./supabase";
import Auth from "./components/Auth.vue";
import Profile from "./components/Profile.vue";
export default {
  components: {
    Auth,
    Profile,
  },

  setup() {
    supabase.auth.getSession().then(({ data }) => {
      store.user = data.session?.user;
    });

    supabase.auth.onAuthStateChange((_, session) => {
      store.user = session?.user;
    });

    return {
      store,
    };
  },
};
</script>
