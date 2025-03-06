<template>
  <div style="width: 100%" class="mx-4">
    <h1 class="text-center mb-4">동아리</h1>

    <v-card elevation="0">
      <v-card-title class="text-h5">새 동아리 만들기</v-card-title>
      <v-card-text>
        <v-text-field
          v-model="newClub.name"
          label="동아리 이름"
          required
        ></v-text-field>

        <v-select
          v-model="newClub.type"
          label="동아리 유형"
          :items="['일반', '벨트', '교사 주도']"
        ></v-select>
      </v-card-text>
      <v-card-actions>
        <v-btn color="primary" @click="createClub">저장</v-btn>
      </v-card-actions>
    </v-card>
  </div>
</template>

<script setup>
const { $db } = useNuxtApp();

const router = useRouter();

const list = ref([]);
const searchByName = ref("");
const searchByMajor = ref([]);
const majors = ref([]);
const onlybelt = ref(false);
const onlyteacher = ref(false);
const dialog = ref(false);

const newClub = ref({
  name: "",
  description: "",
  image: null,
  type: "일반",
  major: [],
});

onMounted(() => {
  const clubRef = dbRef($db, "clubs");
  onValue(clubRef, (snapshot) => {
    const clubs = snapshot.val();
    list.value = clubs;
  });
});

const createClub = async () => {
  // Firebase 저장
  const newClubRef = set(dbRef($db, `clubs/${newClub.value.name}`), {
    name: newClub.value.name,
    description: newClub.value.description,
    image: newClub.value.image
      ? URL.createObjectURL(newClub.value.image)
      : null,
    type: newClub.value.type,
    major: newClub.value.major.map((m) => m.value),
  });

  alert("새 동아리가 생성되었습니다!");
  dialog.value = false;
  newClub.value = {
    name: "",
    description: "",
    image: null,
    type: "일반",
    major: [],
  };

  router.push("/club")
};
</script>

<style>
.club-card {
  width: 150px;
  background-color: #f9f9f9;
  border-radius: 8px;
  overflow: hidden;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.club-card-image {
  background-color: white;
}

.club-card-title {
  text-align: center;
  margin-top: 16px;
  font-size: 12px;
}

.gold-card {
  border: 5px solid gold !important;
}

.green-card {
  border: 5px solid green !important;
}
</style>
