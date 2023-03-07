<script setup lang="ts">
import { ref, computed } from "vue";
import { useFileDialog } from "@vueuse/core";
const { files, open }: any = useFileDialog();
const newKey = ref("");
const signedUrl = ref("");
const uploadResponse: any = ref("");
const loading = ref(false);

const file: any = computed(() =>
  files.value && files.value.length > 0 ? files.value[0] : null
);

const fileType = computed(() => file.value?.type);

const getUploadSignedUrl = async () => {
  loading.value = true;
  const body = {
    key: newKey.value,
    contentType: fileType.value,
  };
  const options = {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: import.meta.env.VITE_API_KEY,
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
const upload = async () => {
  loading.value = true;
  const options: any = {
    method: "PUT",
    headers: { "Content-Type": fileType.value },
    body: file.value,
  };

  const res: any = await fetch(signedUrl.value, options)
    .then((response) => response)
    .catch((err) => console.error(err));
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
    loading
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
