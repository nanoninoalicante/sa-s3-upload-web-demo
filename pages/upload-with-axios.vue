<script setup lang="ts">
import { ref, computed } from "vue";
import { useFileDialog } from "@vueuse/core";
import axios from "axios";

const runtimeConfig = useRuntimeConfig();

const { files, open }: any = useFileDialog();
const newKey = ref("");
const signedUrl = ref("");
const uploadResponse: any = ref("");
const loading = ref(false);
const progressAmount = ref(0);
const progressPercent = computed(
  () => `${Math.round(progressAmount.value * 100)}%`
);

const file: any = computed(() =>
  files.value && files.value.length > 0 ? files.value[0] : null
);

const fileType = computed(() => file.value?.type);

const getUploadSignedUrl = async () => {
  loading.value = true;
  const body = {
    key: newKey.value,
    contentType: fileType.value,
    accelerate: true,
  };
  const options = {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: runtimeConfig.API_KEY || "1234",
    },
    body: JSON.stringify(body),
  };

  const res: any = await fetch(
    "https://dev-upload-s3.staracademyapp.com/get-signed-url",
    options
  )
    .then((response) => response.json())
    .catch((err) => console.error(err));
  signedUrl.value = res.data?.signedUrl;
  loading.value = false;
  return res;
};

const updateProgressAxiosV0 = (event: any) => {
  progressAmount.value = Math.round((event.loaded / event.total) * 100);
};
const updateProgressAxiosV1 = (event: any) => {
  progressAmount.value = event.progress;
};

const upload = async () => {
  loading.value = true;
  const options = {
    method: "PUT",
    url: signedUrl.value,
    headers: { "Content-Type": fileType.value },
    onUploadProgress: (progressEvent: any) =>
      updateProgressAxiosV1(progressEvent),
    data: file.value,
  };
  const res = await axios
    .request(options)
    .then(function (response) {
      console.log(response);
      return response;
    })
    .catch(function (error) {
      console.error(error);
    });

  console.log("res: ", res);
  loading.value = false;
  uploadResponse.value = res;
};
</script>

<template>
  <div
    v-if="loading"
    class="fixed top-0 left-0 bg-white bg-opacity-90 backdrop-blur-lg w-full h-full flex justify-center items-center"
  >
    <div class="flex flex-col justify-center items-center">
      <h1 class="text-2xl font-medium">Uploading</h1>
      <h1 class="text-2xl font-medium">{{ progressPercent }}</h1>
    </div>
  </div>

  <div class="flex flex-col w-full lg:w-2/3">
    <input
      v-model="newKey"
      class="p-6 mb-4 mt-2 rounded-3xl border-2 border-gray-300"
      type="text"
      name="key"
      id="key"
      placeholder="New File Name for S3"
      @keydown.enter="getUploadSignedUrl"
    />
    <button
      @click="getUploadSignedUrl"
      :disabled="!newKey"
      class="rounded-3xl bg-gray-400 hover:bg-blue-400 text-xl p-4 mt-2 mb-12 active:bg-blue-500 disabled:cursor-not-allowed disabled:bg-gray-100 disabled:text-gray-200"
    >
      Get Signed Upload URL
    </button>
    <div v-if="signedUrl" class="flex flex-col w-full space-y-8 mt-4">
      <p>Url: {{ signedUrl }}</p>
      <button
        class="p-4 bg-gray-300 rounded-3xl hover:bg-slate-400"
        type="button"
        @click="open"
      >
        Choose file
      </button>
      <button
        @click="upload"
        class="rounded-3xl bg-gray-400 hover:bg-blue-400 text-xl p-4 my-8 active:bg-blue-500"
      >
        Upload
      </button>
    </div>
    <div class="p-4 my-8" v-for="file in files">
      <p>File Name: {{ file.name }}</p>
      <p>File Type: {{ file.type }}</p>
    </div>
    <div class="p-4 my-8" v-if="uploadResponse">
      <pre>Upload response: {{ uploadResponse.statusText }}</pre>
    </div>
  </div>
</template>
