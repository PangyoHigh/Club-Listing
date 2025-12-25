<template>
  <div v-if="!loggedIn">
    <v-otp-input v-model="tp" length="5"></v-otp-input>
  </div>
  <div v-else class="mx-4">
    <br />

    <div class="mb-8 py-4 rounded-lg" style="border: 1px solid black">
      <h2 class="text-center mb-4">부장/차장 권한</h2>

      <div class="d-flex justify-center ga-5 mb-3">
        <v-btn
          variant="tonal"
          :to="`/clubadmin/message/?clubname=${clubName}`"
          class="text-blue"
        >
          문의 확인하기
        </v-btn>
        <v-btn variant="tonal" @click="changeEmergency" color="red">
          인원부족 상태 {{ clubInfo.emergency ? "해제하기" : "활성화하기" }}
          <v-icon end>mdi-alert</v-icon>
        </v-btn>
      </div>

      <v-text-field
        v-if="clubInfo.emergency"
        v-model="clubInfo.acc"
        variant="outlined"
        label="필요한 인원 수"
      ></v-text-field>

      <div class="d-flex justify-center ga-5 mb-3">
        <v-btn
          v-if="!clubInfo?.finished"
          variant="tonal"
          color="red"
          @click="end"
        >
          수동 마감하기
        </v-btn>
        <v-btn v-else variant="tonal" color="red" @click="undoEnd">
          마감 풀기
        </v-btn>
      </div>
    </div>

    <div class="pa-4 rounded-lg d-flex justify-center">
      <img
        :src="clubInfo?.image ?? '/PGHS.png'"
        draggable="false"
        class="rounded-lg"
        style="width: 300px"
      />
    </div>

    <h1 class="text-center">{{ clubName }}</h1>

    <br />

    <v-textarea
      v-model="clubInfo.description"
      variant="outlined"
      placeholder="동아리 소개글"
    ></v-textarea>

    <div class="d-flex justify-center">
      <v-alert style="display: block; overflow: auto">
        <v-file-input
          accept="image/*"
          label="헤더 이미지 업로드하기"
          variant="underlined"
          @update:model-value="uploadHeader($event)"
        ></v-file-input>

        만일, 이미지를 업로드하지 않는다면, 판교고 로고가 사용될 것입니다.

        <v-img :src="clubInfo?.image" class="rounded-lg mt-3"></v-img>
      </v-alert>
      <br />
      <v-alert>
        <v-file-input
          accept="image/*"
          label="포스터 이미지로 업로드하기"
          variant="underlined"
          @update:model-value="uploadPoster($event)"
        ></v-file-input>

        <v-img :src="clubInfo?.poster" class="rounded-lg"></v-img>
      </v-alert>
    </div>

    <br />

    <v-table style="border: 1px solid black" class="rounded-lg">
      <thead>
        <tr style="background-color: skyblue">
          <th class="text-left">카테고리</th>
          <th class="text-left">정보</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>관련 학과</td>
          <td>
            <v-combobox
              v-model="clubInfo.major"
              :items="majors"
              chips
              closable-chips
              multiple
              variant="outlined"
              class="mt-4"
            >
              <template v-slot:chip="{ props, item }">
                <v-chip v-bind="props" :text="item.raw.name"></v-chip>
              </template>
            </v-combobox>
          </td>
        </tr>
        <tr>
          <td>최대 모집 인원 수</td>
          <td>
            <v-text-field
              v-model="clubInfo.memberNumber"
              variant="outlined"
              class="mt-4"
            ></v-text-field>
          </td>
        </tr>
        <tr>
          <td>모집 방법</td>
          <td>
            <v-select
              v-model="clubInfo.how"
              :items="['면접', '자기소개서', '없음']"
              variant="outlined"
              class="mt-4"
            ></v-select>
          </td>
        </tr>
        <tr>
          <td>지원 방법</td>
          <td>
            <v-autocomplete
              v-model="clubInfo.howmessage"
              :items="['문자', '대면지원', '카카오톡', '인스타', '구글폼']"
              chips
              closable-chips
              multiple
              variant="outlined"
              class="mt-4"
            ></v-autocomplete>

            <div
              v-if="
                clubInfo.howmessage?.includes('문자') ||
                clubInfo.howmessage?.includes('카카오톡')
              "
            >
              <v-text-field
                v-model="clubInfo.phone"
                variant="outlined"
                placeholder="ex) 010-1234-5678"
                class="mt-4"
                label="전화번호"
              ></v-text-field>
            </div>
            <div v-if="clubInfo.howmessage?.includes('대면지원')">
              <v-text-field
                v-model="clubInfo.location"
                variant="outlined"
                placeholder="ex) 2-4반 16번"
                class="mt-4"
                label="반 이름"
              ></v-text-field>
            </div>
            <div v-if="clubInfo.howmessage?.includes('인스타')">
              <v-text-field
                v-model="clubInfo.insta"
                variant="outlined"
                placeholder="ex) @hsl.dev"
                class="mt-4"
                label="인스타 아이디"
              ></v-text-field>
            </div>
            <div v-if="clubInfo.howmessage?.includes('구글폼')">
              <v-text-field
                v-model="clubInfo.formlink"
                variant="outlined"
                placeholder="ex) https://forms.gle/aaaaa"
                class="mt-4"
                label="구글폼 링크"
              ></v-text-field>
            </div>
          </td>
        </tr>
        <tr>
          <td>모집 시기</td>
          <td>
            <v-text-field
              v-model="clubInfo.start"
              variant="outlined"
              placeholder="2024-05-01와 같은 형식으로"
              class="mt-4"
              label="시기"
            ></v-text-field>
            <v-text-field
              v-model="clubInfo.end"
              variant="outlined"
              placeholder="2024-05-16와 같은 형식으로"
              class="mt-4"
              label="종기"
            ></v-text-field>
          </td>
        </tr>
      </tbody>
    </v-table>

    <br />

    <div class="d-flex justify-center">
      <v-btn color="red" @click="update">업데이트</v-btn>
      <v-btn color="black" class="ml-4" @click="deleteClub">삭제</v-btn>
    </div>
  </div>
</template>

<script setup>
import { onAuthStateChanged } from "firebase/auth";
import { remove } from "firebase/database";
import { ref as sRef, uploadBytes, getDownloadURL } from "firebase/storage";

const loggedIn = ref(false);
const tp = ref("");
const clubInfo = ref({
  leader: "",
  coleader: "",
  description: "",
  image: "",
  major: "",
  memberNumber: 20,
  how: "면접",
  start: "2024-05-01",
  end: "2024-05-16",
  getWhyJoined: false,
  howmessage: "",
  phone: "",
  location: "",
  insta: "",
  poster: "",
  acc: 0,
  messaged: 0,
  formlink: "",
  emergency: false,
});
const account = ref({});

const { $auth, $db, $storage } = useNuxtApp();
const route = useRoute();
const router = useRouter();
const clubName = route.query.clubname;

// when tp changed
watch(tp, (v) => {
  if (v === "20255") {
    loggedIn.value = true;
    localStorage.setItem("loggedIn", true);
  }
});

function end() {
  clubInfo.value.finished = true;
  const messageRef = dbRef($db, `clubs/${clubName}`);
  set(messageRef, clubInfo.value);
}

function undoEnd() {
  clubInfo.value.finished = false;
  const messageRef = dbRef($db, `clubs/${clubName}`);
  set(messageRef, clubInfo.value);
}

const majors = [
  "간호학과",
  "건설방재공학과",
  "건축학과",
  "건축공학과",
  "게임학과",
  "경영정보학과",
  "경영학과",
  "경제학과",
  "경찰학과",
  "관광학과",
  "교육학과",
  "국어교육과",
  "국어국문학과",
  "군사학과",
  "기계공학과",
  "기독교학과",
  "노어노문학과",
  "농업경제학과",
  "농업자원경제학과",
  "독어독문학과",
  "동물자원학과",
  "문예창작학과",
  "문헌정보학과",
  "문화재보존학과",
  "물리치료학과",
  "물리학과",
  "법학과",
  "북한학과",
  "불교학과",
  "불어불문학과",
  "사학과",
  "사회학과",
  "사회복지학과",
  "산업공학과",
  "생명과학과",
  "세무학과",
  "서어서문학과",
  "섬유공학과",
  "소방학과",
  "수산생명의학과",
  "수의학과",
  "수학과",
  "심리학과",
  "식품영양학과",
  "신학과",
  "안전공학과",
  "약학과",
  "언어학과",
  "에너지공학과",
  "연극학과",
  "영상학과",
  "영어영문학과",
  "유아교육과",
  "윤리교육과",
  "의학과",
  "인공지능학과",
  "일반사회교육과",
  "일어일문학과",
  "임상병리학과",
  "임상병리학과",
  "자유전공학부",
  "임상병리학과",
  "제과제빵과",
  "임상병리학과",
  "재료공학과",
  "전기전자공학과",
  "정보보안학과",
  "정보통신공학과",
  "정치외교학과",
  "조경학과",
  "조리과학과",
  "중어중문학과",
  "지리학과",
  "지리교육과",
  "지적학과",
  "철도공학과",
  "철학과",
  "치의학과",
  "치위생학과",
  "커뮤니케이션학과",
  "컴퓨터공학과",
  "통계학과",
  "특성화 학과",
  "특수교육과",
  "한문학과",
  "한약학과",
  "한의학과",
  "항공운항학과",
  "행정학과",
  "화학공학과",
  "화학과",
];

onMounted(async () => {
  const clubRef = dbRef($db, `clubs/${clubName}`);
  await onValue(
    clubRef,
    (snapshot) => (clubInfo.value = { ...clubInfo.value, ...snapshot.val() })
  );

  onAuthStateChanged($auth, (user) => {
    account.value = user;
  });

  if (localStorage.getItem("loggedIn")) {
    loggedIn.value = true;
  }
});

const update = () => {
  const clubRef = dbRef($db, `/clubs/${clubName}`);
  set(clubRef, clubInfo.value);
  router.push(`/clubinfo?clubname=${clubName}`);
};

const uploadHeader = (f) => {
  if (!f.type.startsWith("image/")) {
    console.error("File is not an image");
    return;
  }

  const reader = new FileReader();
  reader.readAsDataURL(f);

  reader.onload = (event) => {
    const img = new Image();
    img.src = event.target.result;

    img.onload = () => {
      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d");

      // Define max width and height
      const maxWidth = 500;
      const maxHeight = 500;
      let width = img.width;
      let height = img.height;

      // Resize maintaining aspect ratio
      if (width > maxWidth || height > maxHeight) {
        if (width > height) {
          height *= maxWidth / width;
          width = maxWidth;
        } else {
          width *= maxHeight / height;
          height = maxHeight;
        }
      }

      canvas.width = width;
      canvas.height = height;
      ctx.drawImage(img, 0, 0, width, height);

      canvas.toBlob(
        (blob) => {
          const resizedFile = new File([blob], f.name, {
            type: "image/jpeg",
            lastModified: Date.now(),
          });

          // Upload the resized image
          const storageRef = sRef($storage, `logos/${clubName}`);
          uploadBytes(storageRef, resizedFile).then((snapshot) => {
            getDownloadURL(snapshot.ref).then((downloadURL) => {
              clubInfo.value.image = downloadURL;
            });
          });
        },
        "image/jpeg",
        0.7 // Adjust quality (0.0 - 1.0)
      );
    };
  };
};

const uploadPoster = (f) => {
  console.log(f);

  const maxWidth = 800; // Set your desired max width
  const maxHeight = 800; // Set your desired max height

  const reader = new FileReader();
  reader.readAsDataURL(f);
  reader.onload = (event) => {
    const img = new Image();
    img.src = event.target.result;
    img.onload = () => {
      let width = img.width;
      let height = img.height;

      // Maintain aspect ratio
      if (width > maxWidth || height > maxHeight) {
        const aspectRatio = width / height;
        if (width > height) {
          width = maxWidth;
          height = Math.round(maxWidth / aspectRatio);
        } else {
          height = maxHeight;
          width = Math.round(maxHeight * aspectRatio);
        }
      }

      // Create a canvas to resize the image
      const canvas = document.createElement("canvas");
      canvas.width = width;
      canvas.height = height;
      const ctx = canvas.getContext("2d");

      // Draw resized image
      ctx.drawImage(img, 0, 0, width, height);

      // Convert to Blob
      canvas.toBlob(
        (blob) => {
          const resizedFile = new File([blob], f.name, {
            type: "image/jpeg",
            lastModified: Date.now(),
          });

          // Upload to Firebase Storage
          const storageRef = sRef($storage, `poster/${clubName}`);
          uploadBytes(storageRef, resizedFile).then((snapshot) => {
            getDownloadURL(snapshot.ref).then((downloadURL) => {
              clubInfo.value.poster = downloadURL;
            });
          });
        },
        "image/jpeg",
        0.7
      ); // Adjust quality if needed
    };
  };
};

const deleteClub = () => {
  if (confirm("정말로 삭제하시겠습니까? 이 작업은 되돌릴 수 없습니다.")) {
    const clubRef = dbRef($db, `/clubs/${clubName}`);
    set(clubRef, null)
      .then(() => {
        alert("삭제 완료되었습니다.");
        router.push("/club"); // 홈 페이지로 이동
      })
      .catch((error) => {
        alert("삭제 실패: " + error.message);
      });
  }
};

const changeEmergency = () => {
  clubInfo.value.emergency = !clubInfo.value.emergency;
  const messageRef = dbRef($db, `clubs/${clubName}`);
  set(messageRef, clubInfo.value);
};
</script>
