<template>
  <div style="width: 100%" class="mx-4">
    <div class="d-flex justify-center">
      <h1 class="text-center mb-4">동아리</h1>
    </div>
    <v-btn variant="icon" :to="`/activityadmin/add`"
      ><v-icon>mdi-pencil</v-icon> 수정</v-btn
    >
    <v-text-field
      v-model="searchByName"
      label="동아리 이름으로 검색하기"
      variant="outlined"
      prepend-inner-icon="mdi-magnify"
    ></v-text-field>

    <v-combobox
      v-model="searchByMajor"
      :items="majors"
      chips
      closable-chips
      multiple
      variant="outlined"
      label="학과로 동아리 검색하기"
    >
      <template v-slot:chip="{ props, item }">
        <v-chip v-bind="props" :text="item.name"></v-chip>
      </template>
    </v-combobox>

    <div class="d-flex ga-3">
      <v-checkbox
        color="amber-lighten-1"
        v-model="onlybelt"
        @click="() => (onlyteacher = false)"
        label="벨트 동아리만 보기"
      ></v-checkbox>
      <v-checkbox
        color="green-darken-2"
        v-model="onlyteacher"
        @click="() => (onlybelt = false)"
        label="교사 주도 동아리만 보기"
      ></v-checkbox>
    </div>

    <v-row class="d-flex justify-center">
      <v-col
        v-for="item in Object.values(list ?? {})
          .sort((a, b) => {
            const order = {
              belt: 1,
              none: 2,
              일반: 2,
              teacher: 3,
            };

            return (
              (order[a?.type ?? 'none'] || 4) - (order[b?.type ?? 'none'] || 4)
            );
          })
          .filter((a) => {
            if (onlybelt) {
              return a.type === 'belt';
            } else {
              return true;
            }
          })
          .filter((a) => {
            if (onlyteacher) {
              return a.type === 'teacher';
            } else {
              return true;
            }
          })
          .filter((a) => {
            if (!searchByMajor.length) return true;
            if (searchByMajor) {
              if (!Object.keys(a)?.includes('major')) return false;
              if (searchByMajor.some((r) => a.major?.includes(r.value)))
                return true;
            }
          })
          .filter((a) => {
            if (a.name?.includes(searchByName)) return true;
          })"
        :key="item.name"
        class="d-flex justify-center"
      >
        <v-card
          :to="`/clubinfo?clubname=${item.name}`"
          :class="`club-card ${
            item.type === 'belt'
              ? 'gold-card'
              : item.type === 'teacher'
              ? 'green-card'
              : ''
          }`"
          elevation="4"
        >
          <v-img
            :src="item?.image || item?.poster || '/PGHS.png'"
            class="club-card-image"
            height="150"
            cover
          ></v-img>

          <p class="club-card-title mb-3" style="font-size: 15px">
            {{ item.name }}
          </p>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script setup>
const { $db } = useNuxtApp();

const list = ref([]);
const std = ref("popular");

const onlySeeCanJoin = ref(false);
const onlybelt = ref(false);
const onlyteacher = ref(false);
const searchByName = ref("");
const searchByMajor = ref([]);
const majors = ref([]);

onMounted(() => {
  const clubRef = dbRef($db, "clubs");
  onValue(clubRef, (snapshot) => {
    const clubs = snapshot.val();
    list.value = clubs;

    // 학과별 동아리 개수를 집계
    const majorCounts = {};
    Object.values(clubs ?? {}).forEach((club) => {
      if (club.major) {
        club.major.forEach((major) => {
          if (!majorCounts[major]) {
            majorCounts[major] = 0;
          }
          majorCounts[major]++;
        });
      }
    });

    // majors 목록 생성
    majors.value = Object.keys(majorCounts)
      .sort((a, b) => majorCounts[b] - majorCounts[a])
      .map((major) => ({
        title: `${major} (${majorCounts[major]})`,
        value: major,
      }));
  });
});
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

.club-card-title mark {
  background-color: yellow;
  padding: 2px 4px;
  border-radius: 4px;
}

.club-card-text {
  padding: 8px;
  text-align: center;
  font-size: 14px;
}

.gold-card {
  border: 5px solid gold !important;
}

.green-card {
  border: 5px solid green !important;
}
</style>
