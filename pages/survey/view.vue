<template>
  <div class="page-container mx-4">
    <br /><br />

    <v-card
      variant="tonal"
      prepend-icon="mdi-chevron-left"
      title="다른 동아리 설문 결과"
      subtitle="돌아가기"
      to="/survey/result"
    ></v-card>

    <br /><br />

    <h1 class="text-center club-title">{{ clubName }}</h1>
    <h3 class="text-center club-infos">
      {{ items[clubName].place }} · {{ items[clubName].activity }}
    </h3>

    <div class="d-flex justify-center my-4">
      <v-rating
        :model-value="
          roundRating(listInfo.totalAccumulation, listInfo.totalCount)
        "
        color="amber"
        readonly
        half-increments
        size="x-large"
      ></v-rating>
    </div>

    <v-table class="club-info-table rounded-lg">
      <thead>
        <tr>
          <th class="text-center">카테고리</th>
          <th class="text-center">정보</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>등수</td>
          <td>{{ place }}</td>
        </tr>
        <tr>
          <td>설문 수</td>
          <td>{{ listInfo.totalCount }}명</td>
        </tr>
        <tr>
          <td>평균 점수</td>
          <td>
            {{ roundRating(listInfo.totalAccumulation, listInfo.totalCount) }}점
          </td>
        </tr>
      </tbody>
    </v-table>

    <br /><br />

    <h2 class="text-center reviews-title">후기 보기</h2>
    <div
      v-if="
        Object.values(listInfo.list ?? {}).filter((review) => review.review)
          .length > 0
      "
      class="reviews-section"
    >
      <v-card
        v-for="(review, index) in Object.values(listInfo.list ?? {})"
        v-show="review.review"
        :key="Object.keys(listInfo.list ?? {})[index]"
        variant="tonal"
        class="review-card"
      >
        <template v-slot:prepend>
          <img
            :src="review.photoURL"
            class="review-image mr-3"
            style="outline: 1px solid black"
          />
        </template>
        <template v-slot:title>
          <v-card-subtitle>
            {{ review.name }}
          </v-card-subtitle>
          <v-card-title class="review-text">
            {{ review.review }}
          </v-card-title>
        </template>
      </v-card>
    </div>
    <v-card v-else variant="tonal">
      <v-card-title class="text-center">후기가 아직 없습니다.</v-card-title>
    </v-card>

    <br /><br />

    
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
import { onAuthStateChanged } from "firebase/auth";
import { ref as dbRef, onValue } from "firebase/database";

const clubInfo = ref({
  leader: "",
  coleader: "",
});
const listInfo = ref([]);
const account = ref(null);
const joining = ref({
  name: "",
  email: "",
  photoURL: "",
});
const typeofAccount = ref("");

const { $auth, $db } = useNuxtApp();
const route = useRoute();
const clubName = route.query.clubname;
const place = route.query.place;

const items = {
  무비앤모어: {
    activity: "영화 상영 및 식음료 제공",
    place: "시청각실",
  },
  "요리퐁 조리퐁": {
    activity: "밤티라미수 만들기",
    place: "가사실",
  },
  "축구 동아리": {
    activity: "농구 미니게임",
    place: "운동장",
  },
  "바이오 브릿지": {
    activity: "성남 깃대종 보호 키링 만들기",
    place: "1-1",
  },
  "인공지능 생각": {
    activity: "인공지능 퀴즈, 프로젝트 전시 및 퀴즈",
    place: "1-2",
  },
  방송부: {
    activity: "노래방 + 와플, 핫초코",
    place: "1-3",
  },
  "코드 크래프터": {
    activity: "웹페이지 전시 및 일부 체험",
    place: "1-4",
  },
  심장박동: {
    activity: "타로상담과 향수 제작",
    place: "1-5",
  },
  시네마드림: {
    activity: "영화 촬영본 상영 후 경품 이벤트",
    place: "1-6, 1-7",
  },
  레브: {
    activity: "놀이를 통한 경제 흐름 체험",
    place: "1-8",
  },
  아트캔버스: {
    activity: "타투부스 + 미술 전시",
    place: "2층 홈베이스",
  },
  스타트업: {
    activity: "방탈출 체험 + 퀴즈",
    place: "가온누리실",
  },
  글빛누리: {
    activity: "종합 오락실(보드게임, 책갈피, 보물찾기)",
    place: "도서관",
  },
  "코딩 인싸이트": {
    activity: "게임 개발 결과 체험 + 활동 전시",
    place: "컴퓨터실",
  },
  배구사랑: {
    activity: "배구 경기 베팅 + 스파이크 리시브 체험",
    place: "체육관",
  },
  "빛나는 팀": {
    activity: "댄스부와 게임",
    place: "2-4",
  },
  카이로스: {
    activity: "가치관 기반 교류 행사",
    place: "2-5",
  },
  "픽션 랩": {
    activity: "스토리텔링형 폐교 방탈출",
    place: "2-6",
  },
  티치스트: {
    activity: "카페 체험 (상식 퀴즈, 미션)",
    place: "2-7",
  },
  다이나믹스: {
    activity: "작품 전시와 레이싱 체험",
    place: "2-8",
  },
  아뜰리에: {
    activity: "작품 전시와 레이싱 체험",
    place: "3층 홈베이스",
  },
  네온: {
    activity: "버시킹 공연",
    place: "음악실",
  },
  케미스트: {
    activity: "은하수 에이드 + 열감지 슬라임 만들기",
    place: "과학실 1",
  },
  머큐리: {
    activity: "달고나 만들기 체험 (화학반응 관찰)",
    place: "과학실 2",
  },
};

function roundRating(totalAccumulation, totalCount) {
  return totalCount ? (totalAccumulation / totalCount).toFixed(1) : 0;
}

onMounted(async () => {
  onAuthStateChanged($auth, (user) => {
    account.value = user;
    joining.value.name = user.displayName;
    joining.value.email = user.email;
    joining.value.photoURL = user.photoURL;

    onValue(
      dbRef($db, `everyone/${account.value.displayName}/type`),
      (snapshot) => {
        typeofAccount.value = snapshot.val();
      }
    );
  });

  const clubRef = dbRef($db, `clubs/${clubName}`);
  await onValue(clubRef, (snapshot) => (clubInfo.value = snapshot.val()));

  const listRef = dbRef($db, `clubs/${clubName}`);
  await onValue(listRef, (snapshot) => {
    listInfo.value = snapshot.val();
  });
});
</script>

<style scoped>
.page-container {
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
  padding: 16px;
  border-radius: 8px;
}

.club-title {
  font-size: 24px;
  font-weight: bold;
  margin-top: 16px;
  color: #333;
}

.club-leaders {
  font-size: 16px;
  font-weight: 500;
  color: #555;
  margin-top: 4px;
}

.club-info-table {
  width: 100%;
  margin-top: 20px;
  background-color: #fff;
  border-collapse: collapse;
}

.club-info-table th {
  background-color: skyblue;
  font-weight: bold;
  padding: 8px;
  text-align: center;
  border: 1px solid #ccc;
}

.club-info-table td {
  padding: 8px;
  text-align: center;
  border: 1px solid #ccc;
  color: #555;
}

.reviews-title {
  font-size: 20px;
  font-weight: 500;
  color: #333;
  margin: 24px 0 16px;
}

.reviews-section {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.review-card {
  padding: 12px;
  border-radius: 8px;
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.review-image {
  width: 50px;
  height: 50px;
  border-radius: 50%;
}

.review-text {
  font-size: 16px;
  font-weight: 500;
  color: #333;
}

.review-rating {
  font-size: 14px;
  color: #777;
  margin-top: 4px;
}
</style>
