<script setup lang="ts">
import html2canvas from "html2canvas";
import QrcodeVue from "qrcode.vue";

const printContent = ref<HTMLElement | null>(null);
const isLoading = ref(false);

const image = "image_url";

// Problem: after screenshot got white space even image display normally
// This is useful when working with image with different origin
// Ex: singUrl from aws s3 etc...
async function downloadIMage() {
  const options = {
    allowTaint: true,
    useCORS: true, // to enable cross origin perms
  };

  const originalImage = document.querySelector(
    "#originalImage"
  )! as HTMLImageElement;
  const newImage = document.getElementById("newImage")! as HTMLImageElement;

  // Create canvas for drawing image
  const canvas = document.createElement("canvas");
  const ctx = canvas.getContext("2d");
  canvas.height = originalImage.naturalHeight;
  canvas.width = originalImage.naturalWidth;

  // Drawing original image
  ctx!.drawImage(originalImage, 0, 0);

  // Convert image to base 64
  // Ex: data;...
  const dataUrl = canvas.toDataURL();

  // Set new src newImage with originalImage data base64
  newImage.src = dataUrl;

  try {
    isLoading.value = true;
    await html2canvas(printContent.value as HTMLElement, options).then(
      function (canvas) {
        const link = document.createElement("a");
        link.setAttribute("download", "download.png");
        link.setAttribute(
          "href",
          canvas
            .toDataURL("image/png")
            .replace("image/png", "image/octet-stream")
        );
        link.click();
        isLoading.value = false;
      }
    );
  } catch (error) {
    // Todo: alert warning message
    console.log(error);
  }
}
</script>

<template>
  <div class="relative bg-white container" ref="printContent">
    <div class="mt-10 bg-white z-20 h-screen">
      <div class="py-8 container">
        <div class="flex items-center flex-col">
          <span class="mb-5 text-xl font-semibold"
            >This is useful when working with image with different origin</span
          >

          <!-- originalImage -->
          <img
            v-if="image"
            id="originalImage"
            crossorigin="anonymous"
            class="md:w-36 w-24 h-30 rounded-md"
            :src="image"
            alt="originalImage"
          />

          <!-- newImage -->
          <img id="newImage" class="md:w-36 w-24 h-30 rounded-md" />
        </div>

        <div class="border-dashed border border-gray-500 w-full my-5"></div>
        <div class="flex flex-col justify-center items-center">
          <qrcode-vue
            value="https://github.com/KruyTharin"
            :size="300"
            level="H"
            class="px-4 md:px-0"
          />

          <span class="my-5 text-center">Was this helpful? 👋</span>

          <button
            class="mt-4 w-full md:w-fit"
            :loading="isLoading"
            @click="downloadIMage"
          >
            Save as image
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
