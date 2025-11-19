<script setup>
import { ref } from "vue";

// state
const email = ref("");
const password = ref("");
const user = ref(null); // will hold userId from /api/user
const countries = ref([]);
const message = ref("");

// Hono server base URL
const API = "http://localhost:3000";

// ----------------------
// LOGIN
// ----------------------
async function login() {
  console.log("LOGIN CLICKED", email.value, password.value);
  message.value = "Logging in...";

  try {
    const res = await fetch(`${API}/auth/login`, {
      method: "POST",
      credentials: "include", // send cookies
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        email: email.value,
        password: password.value,
      }),
    });

    console.log("Login response status:", res.status);

    if (!res.ok) {
      const text = await res.text();
      console.error("Login failed:", res.status, text);
      message.value = `Login failed: ${res.status}`;
      return;
    }

    const data = await res.json();
    console.log("Login JSON:", data);

    message.value = data.message || "Logged in!";
    await getUser();
  } catch (err) {
    console.error("Network/login error:", err);
    message.value =
      "Network error: " + (err instanceof Error ? err.message : String(err));
  }
}

// ----------------------
// CHECK USER
// ----------------------
async function getUser() {
  console.log("GET USER CLICKED");
  try {
    const res = await fetch(`${API}/api/user`, {
      credentials: "include",
    });

    console.log("getUser status:", res.status);

    const data = await res.json();
    console.log("getUser JSON:", data);

    if (data.userId) {
      user.value = data.userId;
      message.value = "Logged in!";
    } else {
      user.value = null;
      message.value = data.message || "Not logged in.";
    }
  } catch (err) {
    console.error("getUser error:", err);
    message.value =
      "Error checking user: " +
      (err instanceof Error ? err.message : String(err));
  }
}

// ----------------------
// LOGOUT
// ----------------------
async function logout() {
  console.log("LOGOUT CLICKED");
  try {
    await fetch(`${API}/signout`, {
      method: "GET",
      credentials: "include",
    });

    user.value = null;
    message.value = "Logged out.";
  } catch (err) {
    console.error("Logout error:", err);
    message.value =
      "Error logging out: " +
      (err instanceof Error ? err.message : String(err));
  }
}

// ----------------------
// FETCH COUNTRIES
// ----------------------
async function getCountries() {
  console.log("GET COUNTRIES CLICKED");
  try {
    const res = await fetch(`${API}/countries`, {
      credentials: "include",
    });

    console.log("countries status:", res.status);

    const data = await res.json();
    console.log("countries JSON:", data);

    countries.value = Array.isArray(data) ? data : [];
  } catch (err) {
    console.error("getCountries error:", err);
    message.value =
      "Error loading countries: " +
      (err instanceof Error ? err.message : String(err));
  }
}
</script>

<template>
  <div style="max-width: 450px; margin: 30px auto; font-family: sans-serif">
    <h2>Vue + Hono + Supabase (SSR Cookies)</h2>

    <p>
      <b>{{ message }}</b>
    </p>

    <!-- LOGIN / LOGOUT UI -->
    <div v-if="!user">
      <h3>Login</h3>

      <!-- form is important for password field -->
      <form @submit.prevent="login">
        <input v-model="email" placeholder="email" />
        <br /><br />
        <input v-model="password" type="password" placeholder="password" />
        <br /><br />
        <button type="submit">Login</button>
      </form>
    </div>

    <div v-else>
      <h3>User ID: {{ user }}</h3>
      <button @click="logout">Logout</button>
    </div>

    <hr />

    <!-- ACTION BUTTONS -->
    <button @click="getUser">Check User</button>
    <button @click="getCountries">Load Countries</button>

    <!-- COUNTRIES LIST -->
    <ul>
      <li v-for="c in countries" :key="c.id">
        {{ c.name }}
      </li>
    </ul>
  </div>
</template>

<style>
button {
  cursor: pointer;
  padding: 8px 16px;
  margin-right: 10px;
}
input {
  padding: 6px;
  width: 100%;
}
</style>
