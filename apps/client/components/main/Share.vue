<template>
  <div>
    <dialog
      class="modal mt-[-8vh]"
      :open="shareModalVisible"
    >
      <div
        ref="dialogBoxRef"
        class="modal-box w-[27rem] flex flex-col items-center overflow-hidden"
      >
        <div class="flex">
          <div class="gallery py-2 mr-2">
            <div
              v-for="(imgItem, index) in galleryImgs"
              :key="imgItem.src"
              :class="[
                'border-2 border-transparent gallery-item w-14 mb-2 h-18 cursor-pointer rounded-sm overflow-hidden mr-2',
                {
                  '!border-primary': currImageIndex === index,
                  skeleton: !imgItem.src,
                },
              ]"
              @click="handleSelectImage(index)"
            >
              <img
                v-show="imgItem.src"
                :src="imgItem.src"
                :alt="`Card ${index}`"
              />
            </div>
            <div
              v-if="allowImageUpload"
              class="border-2 border-gray-500 border-dashed gallery-item w-14 mb-2 h-[4.5rem] cursor-pointer rounded-sm overflow-hidden mr-2"
              @click="handleUploadImage"
            >
              <input ref="imageUploadRef" type="file" class="hidden" @change="handleUploadImageChange">
              <button type="button" className="btn btn-ghost rounded-none p-0 h-full w-full hover:bg-gray-200">
                <svg class="h-8 w-8 text-gray-500" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1024 1024"><path fill="currentColor" d="M482 152h60q8 0 8 8v704q0 8-8 8h-60q-8 0-8-8V160q0-8 8-8"/><path fill="currentColor" d="M192 474h672q8 0 8 8v60q0 8-8 8H160q-8 0-8-8v-60q0-8 8-8Z"/></svg>
              </button>
            </div>
          </div>
          <div
            :class="['w-[19rem] h-[27rem]', { skeleton: !shareImageSrc }]"
            ref="imageContainer"
          >
            <img
              v-show="shareImageSrc"
              :src="shareImageSrc"
              alt="Selected Share Image"
              width="400"
              height="600"
              class="rounded-md"
            />
          </div>
        </div>
        <div class="modal-action">
          <button
            class="btn btn-primary"
            @click="copyAndClose"
          >
            复制并关闭
          </button>
          <button
            class="btn"
            @click="hideShareModal"
          >
            关闭
          </button>
        </div>
      </div>
    </dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from "vue";
import { convertTitleToNumber } from "~/composables/main/shareImage/convert";
import {
useGenerateShareImage,
useShareModal,
} from "~/composables/main/shareImage/share";
import { useCourseStore } from "~/store/course";
import { useUserStore } from "~/store/user";
import { getToday } from "~/utils/date";
const courseStore = useCourseStore();
const userStore = useUserStore();
const imageContainer = ref<HTMLDivElement>();

const { shareModalVisible, hideShareModal } = useShareModal();
const {
  generateGalleryImage,
  copyShareImage,
  shareImageSrc,
  galleryImgs,
  clearShareImageSrc,
  handleSelectImage,
  currImageIndex,
  imageUploadRef,
  allowImageUpload,
  handleUploadImage,
  handleUploadImageChange
} = useGenerateShareImage();

watch(shareModalVisible, (newVal) => {
  if (newVal && courseStore.currentCourse?.title) {
    console.log(userStore.user);
    const username = userStore.user?.username || "";
    const convertedTitle = convertTitleToNumber(
      courseStore.currentCourse.title
    );
    const { year, month, day } = getToday();
    generateGalleryImage(convertedTitle, username, `${year}/${month}/${day}`);
  } else {
    clearShareImageSrc();
  }
});

const copyAndClose = () => {
  copyShareImage(currImageIndex.value);
  hideShareModal();
};
</script>
