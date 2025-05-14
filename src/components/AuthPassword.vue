<template>
  <form class="row flex flex-center" @submit.prevent="handleLogin">
    <div class="col-6 form-widget">
      <h1 class="header">Comifuro XX Circle Catalog Dashboard</h1>
      <p class="description">
        Sign in with your email and password which provided in the email we sent
        to your inbox
      </p>
      <div>
        <input
          class="inputField"
          type="email"
          placeholder="Your email"
          v-model="email"
        />
      </div>
      <div>
        <input
          class="inputField"
          type="password"
          placeholder="Your password"
          v-model="password"
        />
      </div>
      <div>
        <input
          type="submit"
          class="button block"
          :value="loading ? 'Loading' : 'Login'"
          :disabled="loading"
        />
      </div>
    </div>
  </form>
</template>

<script>
import { store } from "../store";
import { ref } from "vue";
import { supabase } from "../supabase";

export default {
  setup() {
    const loading = ref(false);
    const email = ref("");
    const password = ref("");

    const handleLogin = async () => {
      try {
        loading.value = true;
        const {
          data: { user },
          error,
        } = await supabase.auth.signInWithPassword({
          email: email.value,
          password: password.value,
        });
        if (error) throw error;
        store.user = user;
      } catch (error) {
        alert(error.error_description || error.message);
      } finally {
        loading.value = false;
      }
    };

    return {
      loading,
      email,
      password,
      handleLogin,
    };
  },
};
</script>
