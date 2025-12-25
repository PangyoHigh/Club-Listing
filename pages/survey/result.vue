<template>
  <div class="club-list-container">
    <br />

    <div class="d-flex justify-center align-center">
      <v-card
        variant="tonal"
        prepend-icon="mdi-clipboard-list"
        title="다른 동아리 평가하기"
        subtitle="동아리 발표회에 참여하세요"
        to="/survey/select"
        width="100%"
      ></v-card>
    </div>

    <br />

    <v-card variant="tonal">
      <v-card-text>
        점수는 동아리가 지금까지 받은 모든 평점을 합한 값입니다. 순위는
        실시간으로 업데이트 됩니다.
      </v-card-text>
    </v-card>

    <br />

    <div class="top-clubs">
      <v-card
        v-for="(item, name) in Object.values(list ?? {})"
        :key="Object.keys(list ?? {})[name]"
        :to="`/survey/view?clubname=${Object.keys(list ?? {})[name]}&place=${
          name + 1
        }`"
        class="club-card top"
        variant="outlined"
        :style="`border-color: ${getColor(name)}`"
      >
        <p
          v-for="item in Object.keys(list ?? {})[name].split(' ')"
          :key="item"
          class="club-name text-h4"
        >
          {{ item }}
        </p>
        <v-rating
          :model-value="roundRating(item.totalAccumulation, item.totalCount)"
          color="amber"
          readonly
          half-increments
          size="small"
          class="rating"
        ></v-rating>
        <v-card-subtitle class="club-info d-flex align-center justify-center ga-4">
          <div>
            <div>
              <span class="label">등수:</span>
              <span class="value"
                ><mark>{{ name + 1 }}등</mark></span
              >
            </div>
            <div>
              <span class="value"
                >총 점수: {{ formatter.format(item.totalAccumulation) }}</span
              >
            </div>
          </div>

          <div>
            <div>
              <span class="value">
                <span class="label">평균 점수:</span>
                <span class="value">{{
                  (item.totalAccumulation / item.totalCount).toFixed(2)
                }}</span>
              </span>
            </div>
            <div>
              <span class="label">설문 수:</span>
              <span class="value">{{ item.totalCount }}</span>
            </div>
          </div>
        </v-card-subtitle>
      </v-card>
    </div>

    <v-footer
      style="
        position: absolute;
        bottom: 0;
        left: 0;
        width: 100vw;
        border-top: 1px solid #e0e0e0;
      "
      class="d-flex flex-column py-1"
    >
      <p>개발자 20416이현승</p>
    </v-footer>
  </div>
</template>

<script setup>
const { $db, $auth } = useNuxtApp();
const router = useRouter();

const list = ref([]);
const club = ref("");
const account = ref(null);

const fetchData = async () => {
  const clubRef = dbRef($db, "clubs");
  const surveyData = await new Promise((resolve) => {
    onValue(clubRef, (snapshot) => resolve(snapshot.val()));
  });

  const clubsRef = dbRef($db, "clubs");
  const clubsData = await new Promise((resolve) => {
    onValue(clubsRef, (snapshot) => resolve(snapshot.val()));
  });

  const sortedEntries = Object.entries(surveyData).sort(([, a], [, b]) => {
    const scoreA = a.totalAccumulation;
    const scoreB = b.totalAccumulation;
    return scoreB - scoreA;
  });

  list.value = Object.fromEntries(sortedEntries);
  club.value = clubsData;
};

const getColor = (rank) => {
  if (rank === 0) return "gold";
  if (rank === 1) return "silver";
  if (rank === 2) return "#CD7F32";
  return "transparent";
};

const formatter = new Intl.NumberFormat("ko-KR", {
  notation: "compact",
  compactDisplay: "short",
});

onMounted(() => {
  fetchData();
});
</script>

<style scoped>
.club-list-container {
  width: 100%;
  padding: 16px;
}

h1 {
  font-size: 20px;
  margin-bottom: 20px;
}

.club-card {
  width: 100%;
  margin-bottom: 16px;
  padding: 8px;
  border-radius: 8px;
  transition: transform 0.2s ease;
}

.club-card.top {
  border: 2px solid;
}

.club-card:hover {
  transform: translateY(-5px);
}

.club-image {
  width: 100%;
  height: 140px;
  border-radius: 4px;
}

.club-name {
  font-size: 13px;
  font-weight: bold;
  text-align: center;
  margin-top: 8px;
}

.rating {
  display: flex;
  justify-content: center;
  margin: 8px 0;
}

.club-info {
  font-size: 14px;
  text-align: center;
  color: #444;
  margin-top: 8px;
  margin-bottom: 15px;
  line-height: 1.4;
}

.club-info .label {
  display: inline-block;
  font-weight: 600;
  color: #666;
  margin-right: 4px;
}

.club-info .value {
  color: #333;
}

.text-h4 {
  font-size: 30px !important;
}
</style>
