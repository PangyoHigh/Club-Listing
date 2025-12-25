<template>
  <div class="page-container mx-4">
    <v-dialog v-model="loading" persistent width="300">
      <v-card class="loading-dialog">
        <v-card-text class="text-center">
          <v-progress-circular
            indeterminate
            color="primary"
            size="50"
          ></v-progress-circular>
          <h3 class="mt-3">로딩중...</h3>
        </v-card-text>
      </v-card>
    </v-dialog>

    <h1 class="text-center club-title">{{ clubName }}</h1>
    <h3 class="text-center club-infos">
      {{ items[clubName].place }} · {{ items[clubName].activity }}
    </h3>

    <div class="review-section text-center">
      <v-rating
        v-model="rating"
        size="x-large"
        half-increments
        color="amber"
      ></v-rating>

      <v-textarea
        v-model="review"
        variant="outlined"
        placeholder="후기를 남겨주세요"
        class="review-textarea"
      ></v-textarea>
      <v-alert class="text-left">
        후기를 남기면 학번, 이름, 후기가 공개됩니다. 평점은 공개되지 않습니다.
      </v-alert>

      <br />

      <v-btn
        v-if="!dialog"
        color="primary"
        block
        rounded
        :disabled="dialog"
        @click="submit"
      >
        제출
      </v-btn>
    </div>

    <v-dialog v-model="dialog" persistent>
      <v-card class="confirmation-dialog">
        <v-card-title class="text-center">설문이 끝났습니다</v-card-title>
        <v-card-text class="dialog-actions">
          <v-card
            variant="tonal"
            prepend-icon="mdi-link"
            title="설문 결과 보기"
            subtitle="다른 동아리 설문 보기"
            to="/survey/result"
          ></v-card>
          <v-card
            variant="tonal"
            prepend-icon="mdi-chevron-left"
            title="돌아가기"
            to="/survey/select"
          ></v-card>
        </v-card-text>
      </v-card>
    </v-dialog>

    <v-dialog v-model="notloggedin" persistent>
      <v-card class="login-dialog">
        <v-card-text class="text-center">
          <v-btn
            @click="login"
            :loading="loading"
            color="red"
            variant="tonal"
            class="flex-grow-1"
            height="48"
          >
            <v-icon start>mdi-google</v-icon> 판교고 구글 계정으로 로그인하기
          </v-btn>
          <v-btn
            @click="loginWithID"
            :loading="loading"
            color="red"
            variant="tonal"
            class="flex-grow-1"
            height="48"
          >
            <v-icon start>mdi-account</v-icon> 학번, 이름으로 로그인하기
          </v-btn>
        </v-card-text>
      </v-card>
    </v-dialog>

    <v-dialog v-model="loginid" persistent>
      <v-card class="login-dialog">
        <v-card-text class="text-center d-flex flex-column gap-4">
          <v-text-field
            v-model="id"
            prepend-inner-icon="mdi-id-card"
            density="compact"
            label="학번 (예: 10101)"
            variant="solo"
            hide-details
            single-line
            class="pb-3"
          ></v-text-field>
          <v-text-field
            v-model="name"
            prepend-inner-icon="mdi-account"
            density="compact"
            label="이름"
            variant="solo"
            hide-details
            single-line
          ></v-text-field>

          <br />

          <div v-if="id && name && !existsStudent(id, name)">
            <h4 class="mb-2">{{ id }}, {{ name }} 학생이 존재하지 않습니다.</h4>
          </div>
          <div v-else-if="id && name && existsStudent(id, name)">
            <h4 class="mb-2">{{ id }}, {{ name }} 학생이 존재합니다.</h4>
            <v-btn
              @click="loginanon(id, name)"
              color="primary"
              block
              variant="tonal"
            >
              계속
            </v-btn>
          </div>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script setup>
import { get } from "firebase/database";
import {
  GoogleAuthProvider,
  signInWithPopup,
  onAuthStateChanged,
  getAuth,
  signInAnonymously,
  updateProfile,
} from "firebase/auth";

const router = useRouter();

const rating = ref(5);
const account = ref(null);
const joining = ref({
  name: "",
  email: "",
  photoURL: "",
});
const typeofAccount = ref("");
const review = ref("");
const dialog = ref(false);
const notloggedin = ref(false);
const loading = ref(false);
const clubData = ref({});
const matched = ref(false);

const loginid = ref(false);
const id = ref("");
const name = ref("");

const { $auth, $db } = useNuxtApp();
const route = useRoute();
const clubName = route.query.clubname;

const auth = getAuth();

function loginanon(id, name) {
  anonymousLoginWithName(`${id} ${name}`);
  notloggedin.value = false;
  loginid.value = false;
}

async function anonymousLoginWithName(name) {
  const result = await signInAnonymously(auth);

  await updateProfile(result.user, {
    displayName: name,
  });

  account.value = result.user;

  console.log("UID:", result.user.uid);
  console.log("Name:", result.user.displayName);
}

function loginWithID() {
  loginid.value = true;
}

function existsStudent(id, name) {
  return students[id] === name;
}

function submit() {
  if (notloggedin.value) return;
  if (dialog.value) {
    alert("설문이 끝났습니다");
  }

  const surveyed = dbRef($db, `clubs/${clubName}/list/${account.value.uid}`);
  set(surveyed, {
    name: joining.value.name,
    email: joining.value?.email,
    photoURL: joining.value?.photoURL,
    rating: rating.value,
    review: review.value,
  });

  const tc = dbRef($db, `clubs/${clubName}/totalCount`);
  const ta = dbRef($db, `clubs/${clubName}/totalAccumulation`);

  get(tc).then((snapshot) => {
    const count = snapshot.val();
    if (count === null) set(tc, 1);
    else set(tc, count + 1);
  });

  get(ta).then((snapshot) => {
    const accumulation = snapshot.val();
    if (accumulation === null) set(ta, rating.value);
    else set(ta, accumulation + rating.value);
  });

  const didit = dbRef($db, `clubs/${clubName}/didSurvey/${account.value.uid}`);
  set(didit, true);

  dialog.value = true;
}

const login = async () => {
  loading.value = true;
  const provider = new GoogleAuthProvider();

  signInWithPopup($auth, provider).then(() =>
    onAuthStateChanged($auth, async (user) => {
      account.value = user;

      if (!user?.email?.includes("pangyo.hs.kr")) {
        loading.value = false;
        alert("판교고 계정이 아닙니다. 꼭 판교고 계정으로 해야 합니다.");
        router.push("/survey/select");
        $auth.signOut();
        return;
      }

      if (user.email === "24050@pangyo.hs.kr") {
        alert("'조현우'님은 설문에 참여하지 못합니다");
        window.location = "https://theannoyingsite.com";
        return;
      }
      if (account.value == null) {
        notloggedin.value = true;
      } else {
        notloggedin.value = false;
        router.go(0);
      }
    })
  );

  setTimeout(() => {
    loading.value = false;
  }, 2000);
};

onMounted(() => {
  onAuthStateChanged($auth, (user) => {
    account.value = user;
    joining.value.name = user?.displayName;
    joining.value.email = user?.email;
    joining.value.photoURL = user?.photoURL;

    if (account.value == null) {
      notloggedin.value = true;
    }

    onValue(
      dbRef($db, `everyone/${account.value?.displayName}/type`),
      (snapshot) => {
        typeofAccount.value = snapshot.val();
      }
    );

    const didit = dbRef(
      $db,
      `clubs/${clubName}/didSurvey/${account.value?.uid}`
    );
    onValue(didit, (snapshot) => {
      if (snapshot.val() === true) {
        dialog.value = true;
      }
    });

    if (
      checkIfMember(account.value.displayName) === clubName ||
      clubName.includes(checkIfMember(account.value.displayName))
    ) {
      matched.value = true;
    } else {
      matched.value = false;
    }
  });

  const clubInfo = dbRef($db, `clubs/${clubName}`);
  onValue(clubInfo, (snapshot) => {
    clubData.value = snapshot.val();
  });
});

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
const students = {
  10101: "고연서",
  10102: "김세빈",
  10103: "김소율",
  10104: "김수빈",
  10105: "김원",
  10106: "김재연",
  10107: "김지원",
  10108: "김진영",
  10109: "김현준",
  10110: "나연후",
  10111: "박루나",
  10112: "박주하",
  10113: "신예주",
  10114: "신재이",
  10115: "유승원",
  10116: "이선우",
  10117: "이시원",
  10118: "이하린",
  10119: "장규언",
  10120: "정민권",
  10121: "정채원",
  10122: "차준우",
  10123: "최지훈",
  10124: "최희진",
  10125: "한담희",
  10126: "홍성준",
  10201: "권강우",
  10202: "김도영",
  10203: "김민관",
  10204: "김민서",
  10205: "김아린",
  10206: "김재윤",
  10207: "김준범",
  10208: "김하영",
  10209: "김효빈",
  10210: "배효빈",
  10211: "서연우",
  10212: "심채은",
  10213: "엄시우",
  10214: "윤휘성",
  10215: "이유진",
  10216: "이정진",
  10217: "이혜민",
  10218: "임태윤",
  10219: "전빛나",
  10220: "정애린",
  10221: "진가인",
  10222: "최고은",
  10223: "최윤호",
  10224: "최은수",
  10225: "한상혁",
  10226: "한유진",
  10301: "권민석",
  10302: "김경률",
  10303: "김고은",
  10304: "김연희",
  10305: "김예린",
  10306: "김재현",
  10307: "문다몬",
  10308: "민건우",
  10309: "박서윤",
  10310: "박정민",
  10311: "박하민",
  10312: "서하람",
  10313: "석세영",
  10314: "신준하",
  10315: "양시호",
  10316: "오승윤",
  10317: "윤민상",
  10318: "이건희",
  10319: "이연교",
  10320: "장예나",
  10321: "장주하",
  10322: "정유진",
  10323: "정한나",
  10324: "조은결",
  10325: "천소은",
  10326: "강건우",
  10401: "강무연",
  10402: "강윤서",
  10403: "국재영",
  10404: "김도윤",
  10405: "김민재",
  10406: "김시후",
  10407: "김예빈",
  10408: "김정윤",
  10409: "김태리",
  10410: "박채은",
  10411: "서준우",
  10412: "안서진",
  10413: "안수아",
  10414: "양지훈",
  10415: "이노아",
  10416: "이동명",
  10417: "이수민",
  10418: "이예진",
  10419: "이예함",
  10420: "이정준",
  10421: "이현준",
  10422: "인지후",
  10423: "전예찬",
  10424: "차정우",
  10425: "최유선",
  10426: "황현준",
  10501: "강지원",
  10502: "김도영",
  10503: "김도은",
  10504: "김민찬",
  10505: "김열매",
  10506: "김효린",
  10507: "문승원",
  10508: "박선민",
  10509: "안수인",
  10510: "안주성",
  10511: "양주혁",
  10512: "유건우",
  10513: "유준현",
  10514: "이규원",
  10515: "이다경",
  10516: "이송하",
  10517: "이시현",
  10518: "이효훈",
  10519: "인지성",
  10520: "장재영",
  10521: "정진우",
  10522: "정채이",
  10523: "지은찬",
  10524: "최승훈",
  10525: "최유진",
  10526: "한정원",
  10601: "강지호",
  10602: "김동혁",
  10603: "김수형",
  10604: "김유건",
  10605: "김윤후",
  10606: "김재윤",
  10607: "박서이",
  10608: "박승원",
  10609: "박윤민",
  10610: "백지희",
  10611: "백채민",
  10612: "송가은",
  10613: "신재이",
  10614: "신지우",
  10615: "유준상",
  10616: "윤서영",
  10617: "이우민",
  10618: "이준석",
  10619: "이지우",
  10620: "인하영",
  10621: "장유리",
  10622: "정진",
  10623: "차동빈",
  10624: "피호은",
  10625: "황하은",
  10701: "강민진",
  10702: "강윤재",
  10703: "권지우",
  10704: "김가은",
  10705: "김나현",
  10706: "김도은",
  10707: "김준영",
  10708: "김태윤",
  10709: "노서현",
  10710: "마준환",
  10711: "문정원",
  10712: "박보윤",
  10713: "박주원",
  10714: "유서영",
  10715: "윤성민",
  10716: "이상연",
  10717: "이준영",
  10718: "이지유",
  10719: "이혜민",
  10720: "이효진",
  10721: "임연지",
  10722: "정수범",
  10723: "정시환",
  10724: "정영훈",
  10725: "최성진",
  10801: "권민성",
  10802: "김현욱",
  10803: "남지훈",
  10804: "박규민",
  10805: "박시은",
  10806: "박지민",
  10807: "백종훈",
  10808: "서현진",
  10809: "송영운",
  10810: "송하람",
  10811: "심예원",
  10812: "이명준",
  10813: "이상민",
  10814: "이송연",
  10815: "이은찬",
  10816: "이하람",
  10817: "장준기",
  10818: "전수빈",
  10819: "정서현",
  10820: "조은서",
  10821: "최동식",
  10822: "최효제",
  10823: "하동근",
  10824: "홍예린",
  10825: "황예림",
  20101: "강나율",
  20102: "곽도희",
  20103: "김기은",
  20104: "김나현",
  20105: "김다온",
  20106: "김준원",
  20107: "나한빈",
  20108: "문효은",
  20109: "민서정",
  20110: "박지윤",
  20111: "송윤아",
  20112: "엄의진",
  20113: "왕윤",
  20114: "유도영",
  20115: "유의선",
  20116: "이수연",
  20117: "이재원",
  20118: "이준휘",
  20119: "임동건",
  20120: "임샤론",
  20121: "장정원",
  20122: "진승찬",
  20123: "최수인",
  20124: "최수현",
  20125: "최호영",
  20201: "강예준",
  20202: "권효재",
  20203: "김기림",
  20204: "김민재",
  20205: "김시영",
  20206: "김유빈",
  20207: "김해준",
  20208: "김현서",
  20209: "박가온",
  20210: "박시후",
  20211: "박주하",
  20212: "박휘재",
  20213: "신지훈",
  20214: "안소현",
  20215: "안태준",
  20216: "오승헌",
  20217: "유현의",
  20218: "이나리",
  20219: "이연서",
  20220: "이현준",
  20221: "임예준",
  20222: "정세진",
  20223: "조한기",
  20224: "차도연",
  20225: "최해린",
  20301: "고서영",
  20302: "곽민기",
  20303: "김규린",
  20304: "김도윤",
  20305: "김리원",
  20306: "김민석",
  20307: "김서연",
  20308: "김지완",
  20309: "김현서",
  20310: "김현종",
  20311: "김희성",
  20312: "마가인",
  20313: "박준서",
  20314: "배강유",
  20315: "서진웅",
  20316: "오연학",
  20317: "오유근",
  20318: "유예민",
  20319: "이시우",
  20320: "이유찬",
  20321: "전은채",
  20322: "조민규",
  20323: "조유나",
  20324: "조윤서",
  20325: "홍지민",
  20401: "김동겸",
  20402: "남은수",
  20403: "박성현",
  20404: "박준성",
  20405: "박지완",
  20406: "백상기",
  20407: "송하온",
  20408: "오승현",
  20409: "오현재",
  20410: "윤지환",
  20411: "이상우",
  20412: "이서은",
  20413: "이성현",
  20414: "이준성",
  20415: "이채원",
  20416: "이현승",
  20417: "임예건",
  20418: "임윤찬",
  20419: "전예성",
  20420: "정기훈",
  20421: "정상휘",
  20422: "정서윤",
  20423: "정인용",
  20424: "조은산",
  20425: "한채원",
  20501: "곽용현",
  20502: "권대환",
  20503: "김도경",
  20504: "김도현",
  20505: "김원진",
  20506: "김한별",
  20507: "나은혁",
  20508: "마다인",
  20509: "박미주",
  20510: "박준영",
  20511: "박찬휘",
  20512: "오한별",
  20513: "왕시현",
  20514: "유현성",
  20515: "윤영민",
  20516: "이서원",
  20517: "이정인",
  20518: "이지유",
  20519: "장윤석",
  20520: "조민준",
  20521: "조윤진",
  20522: "조현우",
  20523: "지성윤",
  20524: "한찬희",
  20525: "함형준",
  20601: "강태윤",
  20602: "김아람",
  20603: "김욱",
  20604: "김은우",
  20605: "김지효",
  20606: "김혜리",
  20607: "문주하",
  20608: "백규태",
  20609: "서예림",
  20610: "심하은",
  20611: "안서연",
  20612: "오시은",
  20613: "유승민",
  20614: "윤지성",
  20615: "이지호",
  20616: "임동연",
  20617: "전지민",
  20618: "정혜원",
  20619: "조한나",
  20620: "최유진",
  20621: "한지민",
  20622: "허윤서",
  20623: "허지우",
  20624: "홍지우",
  20625: "양찬",
  20626: "NaN",
  20701: "강미소",
  20702: "고은재",
  20703: "곽예랑",
  20704: "김래하",
  20705: "김서윤",
  20706: "김시윤",
  20707: "김예진",
  20708: "김용우",
  20709: "김준현",
  20710: "박수진",
  20711: "서민재",
  20712: "신이현",
  20713: "이현서",
  20714: "이혜리",
  20715: "장준봉",
  20716: "장현지",
  20717: "전지윤",
  20718: "정하랑",
  20719: "최승비",
  20720: "최윤서",
  20721: "표가은",
  20722: "한혜빈",
  20723: "허예린",
  20724: "허지수",
  20725: "황채원",
  20801: "김나현",
  20802: "김서준",
  20803: "김시온",
  20804: "김연서",
  20805: "김재현",
  20806: "김태은",
  20807: "문호준",
  20808: "박규림",
  20809: "안석우",
  20810: "원윤재",
  20811: "유영현",
  20812: "이서진",
  20813: "이석희",
  20814: "이우진",
  20815: "이진하",
  20816: "이창빈",
  20817: "이태경",
  20818: "이태현",
  20819: "이한나",
  20820: "정지원",
  20821: "정현제",
  20822: "조재형",
  20823: "최준호",
  20824: "한서진",
  20825: "황유찬",
};
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
  font-size: 30px;
  font-weight: bold;
  margin-top: 16px;
  color: #333;
}

.club-infos {
  font-size: 16px;
  font-weight: 500;
  color: #555;
  margin-top: 4px;
}
.review-section {
  margin-top: 20px;
}

.review-textarea {
  width: 100%;
  margin-top: 12px;
  background-color: #fff;
  border-radius: 4px;
}

.v-btn {
  margin-top: 16px;
}

.confirmation-dialog,
.login-dialog {
  padding: 16px;
  background-color: #fff;
  border-radius: 8px;
}

.dialog-actions {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.user-info-table td {
  padding: 8px;
  color: #333;
}

.v-btn[block] {
  width: 100%;
  margin-top: 8px;
}
</style>
