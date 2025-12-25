<template>
  <div class="mx-4">
    <div>
      <br /><br />

      <div class="d-flex justify-center align-center">
        <div class="text-center">
          <v-img
            src="https://pangyo-lufyc.web.app/PGHS.png"
            width="75"
            class="ma-auto mb-2"
          ></v-img>
          <p>판교고 동아리 발표회</p>
        </div>
      </div>

      <br />

      <v-autocomplete
        :items="items"
        density="comfortable"
        menu-icon=""
        placeholder="이름으로 검색하기"
        prepend-inner-icon="mdi-magnify"
        style="width: calc(100vw - 100px); margin-top: 10px"
        theme="light"
        variant="outlined"
        auto-select-first
        item-props
        rounded
        no-data-text="검색 결과가 없습니다."
      ></v-autocomplete>
    </div>

    <v-footer
      style="
        position: fixed;
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
const items = [
  {
    title: "무비앤모어",
    activity: "영화 상영 및 식음료 제공",
    place: "시청각실",
    to: "/survey/club/?clubname=무비앤모어",
  },
  {
    title: "요리퐁 조리퐁",
    activity: "밤티라미수 만들기",
    place: "가사실",
    to: "/survey/club/?clubname=요리퐁 조리퐁",
  },
  {
    title: "축구 동아리",
    activity: "농구 미니게임",
    place: "운동장",
    to: "/survey/club/?clubname=축구 동아리",
  },
  {
    title: "바이오 브릿지",
    activity: "성남 깃대종 보호 키링 만들기",
    place: "1-1",
    to: "/survey/club/?clubname=바이오 브릿지",
  },
  {
    title: "인공지능 생각",
    activity: "인공지능 퀴즈, 프로젝트 전시 및 퀴즈",
    place: "1-2",
    to: "/survey/club/?clubname=인공지능 생각",
  },
  {
    title: "방송부",
    activity: "노래방 + 와플, 핫초코",
    place: "1-3",
    to: "/survey/club/?clubname=방송부",
  },
  {
    title: "코드 크래프터",
    activity: "웹페이지 전시 및 일부 체험",
    place: "1-4",
    to: "/survey/club/?clubname=코드 크래프터",
  },
  {
    title: "심장박동",
    activity: "타로상담과 향수 제작",
    place: "1-5",
    to: "/survey/club/?clubname=심장박동",
  },
  {
    title: "시네마드림",
    activity: "영화 촬영본 상영 후 경품 이벤트",
    place: "1-6, 1-7",
    to: "/survey/club/?clubname=시네마드림",
  },
  {
    title: "레브",
    activity: "놀이를 통한 경제 흐름 체험",
    place: "1-8",
    to: "/survey/club/?clubname=레브",
  },
  {
    title: "아트캔버스",
    activity: "타투부스 + 미술 전시",
    place: "2층 홈베이스",
    to: "/survey/club/?clubname=아트캔버스",
  },
  {
    title: "스타트업",
    activity: "방탈출 체험 + 퀴즈",
    place: "가온누리실",
    to: "/survey/club/?clubname=스타트업",
  },
  {
    title: "글빛누리",
    activity: "종합 오락실(보드게임, 책갈피, 보물찾기)",
    place: "도서관",
    to: "/survey/club/?clubname=글빛누리",
  },
  {
    title: "코딩 인싸이트",
    activity: "컴퓨터실",
    place: "게임 개발 결과 체험 + 활동 전시",
    to: "/survey/club/?clubname=코딩 인싸이트",
  },
  {
    title: "배구사랑",
    activity: "배구 경기 베팅 + 스파이크 리시브 체험",
    place: "체육관",
    to: "/survey/club/?clubname=배구사랑",
  },
  {
    title: "빛나는 팀",
    activity: "댄스부와 게임",
    place: "2-4",
    to: "/survey/club/?clubname=빛나는 팀",
  },
  {
    title: "카이로스",
    activity: "가치관 기반 교류 행사",
    place: "2-5",
    to: "/survey/club/?clubname=카이로스",
  },
  {
    title: "픽션 랩",
    activity: "스토리텔링형 폐교 방탈출",
    place: "2-6",
    to: "/survey/club/?clubname=픽션 랩",
  },
  {
    title: "티치스트",
    activity: "카페 체험 (상식 퀴즈, 미션)",
    place: "2-7",
    to: "/survey/club/?clubname=티치스트",
  },
  {
    title: "다이나믹스",
    activity: "작품 전시와 레이싱 체험",
    place: "2-8",
    to: "/survey/club/?clubname=다이나믹스",
  },
  {
    title: "아뜰리에",
    activity: "작품 전시와 레이싱 체험",
    place: "3층 홈베이스",
    to: "/survey/club/?clubname=아뜰리에",
  },
  {
    title: "네온",
    activity: "버시킹 공연",
    place: "음악실",
    to: "/survey/club/?clubname=네온",
  },
  {
    title: "케미스트",
    activity: "은하수 에이드 + 열감지 슬라임 만들기",
    place: "과학실 1",
    to: "/survey/club/?clubname=케미스트",
  },
  {
    title: "머큐리",
    activity: "달고나 만들기 체험 (화학반응 관찰)",
    place: "과학실 2",
    to: "/survey/club/?clubname=머큐리",
  },
];
</script>

<style scoped></style>
