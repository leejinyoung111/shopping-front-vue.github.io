<script setup>
import { AddCartApi } from "@/api/cart";
import { kakaoKey } from "@/constants/envName";
import { priceChange } from "@/utils/PriceConversion";
import axios from "axios";
import { onMounted, ref } from "vue";
import { useAuthStore } from "@/stores/auth";
import ContainerLayout from "@/components/layout/ContainerLayout.vue";
import RedButton from "@/components/button/RedButton.vue";
import WhiteButton from "@/components/button/WhiteButton.vue";
import { toastAlert } from "@/utils/ToastAlert";

// storage
const authStore = useAuthStore();

// 변수
const url = window.location.pathname;
const param = url.slice(13, url.length);
const bookDetailInfo = ref();
const getToken = ref(JSON.parse(localStorage.getItem("accessToken")));
const getUser = ref();
const bookCount = ref(1);

// 유저 정보 가져오기
const getUserInfo = async () => {
  if (getToken.value != null) {
    // 토큰으로 유저 정보 가져오기
    const user = await authStore.getUserInfo(getToken.value);
    getUser.value = user.userInfo.user;
  }
};

// 도서 디테일 정보 가져오기
const getBookInfo = async (param) => {
  try {
    const result = await axios.get(
      `https://dapi.kakao.com/v3/search/book?target=isbn&query=${param}`,
      {
        headers: {
          Authorization: `KakaoAK ${kakaoKey}`,
        },
      }
    );

    if (result.status == 200) {
      bookDetailInfo.value = result.data.documents[0];
    } else {
      console.log("api 호출 에러 : " + result);
    }
  } catch (e) {
    console.log(e);
  }
};

// 수량 변경
const changeCount = (status) => {
  if (status == "minus") {
    if (bookCount.value > 1) {
      bookCount.value -= 1;
    }
  } else if (status == "plus") {
    if (bookCount.value < 5) {
      bookCount.value += 1;
    }
  }
};

// 장바구니 추가
const addCart = async (bookDetailInfo) => {
  try {
    const { title, price, thumbnail, publisher } = bookDetailInfo;
    const value = {
      user_id: getUser.value.id,
      book_id: param,
      title,
      price,
      thumb_nail: thumbnail,
      publisher,
      count: bookCount.value,
    };

    const result = await AddCartApi(value);

    const status = result.data.status;

    toastAlert({ message: status.message, toastType: status.status });
  } catch (e) {
    console.log(e);
  }
};

onMounted(() => {
  getUserInfo();
  getBookInfo(param);
});
</script>
<template>
  <ContainerLayout>
    <div v-if="bookDetailInfo != undefined" class="container p-5 mx-auto">
      <div
        class="lg:w-4/5 h-full flex flex-col lg:flex-row justify-center items-center"
      >
        <img
          class="w-2/3 md:w-1/2 lg:w-2/5 object-cover object-center rounded"
          :src="`${bookDetailInfo.thumbnail}`"
          :alt="bookDetailInfo.title"
        />

        <!-- 디테일 정보 -->
        <div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
          <h1 class="text-gray-900 text-2xl title-font font-medium mb-1">
            {{ bookDetailInfo.title }}
          </h1>
          <h1 class="text-gray-700 text-md title-font font-medium mb-1">
            출판사 : {{ bookDetailInfo.publisher }}
          </h1>
          <h1 class="text-gray-700 text-md title-font font-medium mb-1">
            저자 : {{ bookDetailInfo.authors.join(", ") }}
          </h1>
          <p class="leading-relaxed">
            {{ bookDetailInfo.contents }}
          </p>
          <div
            class="flex mt-6 items-center pb-5 border-b-2 border-gray-200 mb-5"
          ></div>
          <div class="flex flex-col gap-2">
            <h1 class="text-gray-700 text-md title-font font-medium mb-1">
              판매 상태 : {{ bookDetailInfo.status }}
            </h1>
            <span class="title-font font-medium text-xl text-gray-900"
              >정가 : {{ priceChange(bookDetailInfo.price) }}원</span
            >

            <span
              v-if="getUser && getUser.role == 'user'"
              class="flex gap-5 py-1 items-center title-font font-medium text-xl text-gray-900"
              >수량 :
              <div class="relative flex items-center max-w-[8rem]">
                <button
                  type="button"
                  class="bg-gray-100 dark:bg-gray-700 dark:hover:bg-gray-600 dark:border-gray-600 hover:bg-gray-200 border border-gray-300 rounded-s-lg p-3 h-8 flex justify-center items-center focus:ring-gray-100 dark:focus:ring-gray-700 focus:ring-2 focus:outline-none"
                  :class="
                    bookCount == 1 && 'cursor-not-allowed hover:bg-gray-100'
                  "
                  @click="changeCount('minus')"
                >
                  <svg
                    class="w-3 h-3 text-gray-900 dark:text-white"
                    aria-hidden="true"
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 18 2"
                  >
                    <path
                      stroke="currentColor"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M1 1h16"
                    />
                  </svg>
                </button>
                <input
                  type="text"
                  v-model="bookCount"
                  class="bg-gray-50 border-x-0 border-gray-300 h-8 text-center text-gray-900 text-sm focus:ring-blue-500 focus:border-blue-500 block w-full py-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                  readonly
                  required
                />
                <button
                  type="button"
                  class="bg-gray-100 dark:bg-gray-700 dark:hover:bg-gray-600 dark:border-gray-600 hover:bg-gray-200 border border-gray-300 rounded-e-lg p-3 h-8 flex justify-center items-center focus:ring-gray-100 dark:focus:ring-gray-700 focus:ring-2 focus:outline-none"
                  :class="
                    bookCount == 5 && 'cursor-not-allowed hover:bg-gray-100'
                  "
                  @click="changeCount('plus')"
                >
                  <svg
                    class="w-3 h-3 text-gray-900 dark:text-white"
                    aria-hidden="true"
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 18 18"
                  >
                    <path
                      stroke="currentColor"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M9 1v16M1 9h16"
                    />
                  </svg>
                </button>
              </div>
            </span>

            <!-- 장바구니 -->
            <div
              v-if="getUser && getUser.role == 'user'"
              class="flex justify-start items-center gap-5"
            >
              <RedButton
                v-if="bookDetailInfo.status == '정상판매'"
                type="button"
                text="장바구니"
                add-class="w-2/3 md:w-2/4 lg:w-1/3  my-3"
                @click="addCart(bookDetailInfo)"
              />

              <WhiteButton
                v-else
                type="button"
                text="품절"
                add-class="w-2/3 md:w-2/4 lg:w-1/3 my-3 cursor-not-allowed bg-gray-200 hover:bg-gray-200"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </ContainerLayout>
</template>
