<template>
  <v-app>
    <VitePwaManifest />
    <NuxtLoadingIndicator />

    <v-app-bar :elevation="0" v-if="!route.path.includes('survey')">
      <template v-slot:prepend>
        <v-app-bar-nav-icon @click="appbar = !appbar"></v-app-bar-nav-icon>
      </template>

      <v-app-bar-title>
        <NuxtLink to="/" class="text-decoration-none text-black">찾동</NuxtLink>
      </v-app-bar-title>
    </v-app-bar>

    <v-navigation-drawer v-model="appbar" v-if="!route.path.includes('survey')">
      <v-list density="compact">
        <v-list-item
          title="찾아줄게! 너의 동아리"
          subtitle="코딩인싸이트"
        ></v-list-item>
        <v-divider class="my-3"></v-divider>
        <v-list-item color="primary" rounded="xl" to="/">
          <v-list-item-title
            ><v-icon start>mdi-home</v-icon> 홈</v-list-item-title
          >
        </v-list-item>
        <v-list-item color="primary" rounded="xl" to="/club">
          <v-list-item-title
            ><v-icon start>mdi-account-group</v-icon> 동아리</v-list-item-title
          >
        </v-list-item>
        <v-list-item color="primary" rounded="xl" to="/write">
          <v-list-item-title
            ><v-icon start>mdi-pencil-box</v-icon> 활동기록</v-list-item-title
          >
        </v-list-item>
        <v-list-item color="primary" rounded="xl" to="/recommend">
          <v-list-item-title
            ><v-icon start>mdi-robot-outline</v-icon> 동아리
            추천</v-list-item-title
          >
        </v-list-item>
        <v-list-item color="primary" rounded="xl" to="/account">
          <v-list-item-title
            ><v-icon start>mdi-account</v-icon> 계정</v-list-item-title
          >
        </v-list-item>
        <v-list-item
          v-if="typeofAccount === 'teacher'"
          color="primary"
          rounded="xl"
          to="/admin"
        >
          <v-list-item-title
            ><v-icon start>mdi-account-lock-outline</v-icon> 관리자
            페이지</v-list-item-title
          >
        </v-list-item>

        <v-divider class="my-3"></v-divider>

        <v-list-item color="primary" rounded="xl" to="/survey/select">
          <v-list-item-title>
            <v-icon start>mdi-clipboard</v-icon> 동아리 평가하기
          </v-list-item-title>
        </v-list-item>
        <v-list-item color="primary" rounded="xl" to="/survey/result">
          <v-list-item-title>
            <v-icon start>mdi-clipboard-list</v-icon> 동아리 발표회
            결과보기</v-list-item-title
          >
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <NuxtLayout>
      <v-main>
        <div class="d-flex justify-center">
          <NuxtPage />
        </div>

        <template v-if="route.path !== '/recommend' && route.path !== '/'">
          <br /><br /><br />
        </template>
      </v-main>
    </NuxtLayout>
  </v-app>
</template>

<script setup>
import { onAuthStateChanged } from "firebase/auth";

const { $auth, $db } = useNuxtApp();

const router = useRouter();
const route = useRoute();

const appbar = ref(false);

const account = ref({});
const typeofAccount = ref("member");

onMounted(() => {
  onAuthStateChanged($auth, (user) => {
    account.value = user;

    if (account.value === null) {
      return;
    }

    const type = dbRef($db, `/everyone/${user?.displayName}/type`);
    onValue(type, (snapshot) => (typeofAccount.value = snapshot.val()));
  });

  // if (process.client) {
  //   window.CHIPP_APP_URL = "https://-46271.chipp.ai";
  //   window.CHIPP_APP_ID = 46271;

  //   const link = document.createElement("link");
  //   link.rel = "stylesheet";
  //   link.href =
  //     "https://storage.googleapis.com/chipp-chat-widget-assets/build/bundle.css";
  //   document.head.appendChild(link);

  //   const script = document.createElement("script");
  //   script.src =
  //     "https://storage.googleapis.com/chipp-chat-widget-assets/build/bundle.js";
  //   script.defer = true;
  //   document.body.appendChild(script);
  // }
});

useHead({
  title: "찾아줄게! 너의 동아리",
});
</script>
