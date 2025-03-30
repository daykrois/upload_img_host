<template>
  <div class="flex flex-col min-h-screen bg-gray-50 justify-center items-center p-4">
    <h1 class="text-3xl font-bold text-gray-800 mb-8">提交图片到GitHub</h1>
    <form
      @submit.prevent="uploadImg"
      class="w-full max-w-md space-y-6 bg-white p-8 rounded-lg shadow-md"
    >
      <div class="space-y-2">
        <label class="block text-sm font-medium text-gray-700">用户名</label>
        <input
          type="text"
          v-model.trim="form.owner"
          required
          class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500"
        />
      </div>
      <div class="space-y-2">
        <label class="block text-sm font-medium text-gray-700">仓库名</label>
        <input
          type="text"
          required
          v-model.trim="form.repo"
          class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500"
        />
      </div>
      <div class="space-y-2">
        <label class="block text-sm font-medium text-gray-700">路径</label>
        <input
          type="text"
          required
          v-model.trim="form.path"
          class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500"
        />
      </div>
      <div class="space-y-2">
        <label class="block text-sm font-medium text-gray-700">GitHub PAT</label>
        <input
          type="password"
          v-model.trim="form.pat"
          required
          class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500"
        />
      </div>
      <div
        class="flex flex-wrap flex-col py-10 items-center justify-center border-2 border-dashed hover:border-blue-400 hover:bg-blue-50 border-gray-300 rounded-lg"
      >
        <div class="text-5xl mb-4 text-blue-500">📁</div>
        <!-- <button
          class="bg-blue-500 text-white px-5 py-2 rounded-md hover:bg-blue-600 transition-colors"
          @click="fileInput.click()"
        >
          选择图片
        </button> -->
        <label
          for="fileInput"
          class="bg-blue-500 text-white px-5 py-2 rounded-md hover:bg-blue-600 transition-colors cursor-pointer"
        >
          选择图片
        </label>
        <input
          id="fileInput"
          ref="fileInput"
          type="file"
          name="img"
          accept="image/*"
          @change="changeImg"
          required
          hidden
        />
      </div>
      <img v-if="previewUrl" :src="previewUrl" alt="" />

      <button
        type="submit"
        class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
      >
        提交
      </button>
    </form>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

interface FormData {
  owner: string
  repo: string
  path: string
  pat: string
  img: string | null
}

const previewUrl = ref<string>('')

const fileInput = ref<HTMLInputElement>(null!)

const form = ref<FormData>({
  owner: '',
  repo: '',
  path: '',
  pat: '',
  img: null,
})

const changeImg = async () => {
  const file = fileInput.value.files?.[0]

  if (file) {
    const reader = new FileReader()
    reader.readAsDataURL(file)
    let img_base64 = ''
    reader.onload = () => {
      if (typeof reader.result === 'string') {
        img_base64 = reader.result.split(',')[1]
        form.value.img = img_base64
        console.log(form.value.img)
      }
    }
    previewUrl.value = URL.createObjectURL(file)
    // console.log(btoa(String.fromCharCode(...new Uint8Array(await file.arrayBuffer()))))
  }
}

const uploadImg = async () => {
  const file = fileInput.value.files?.[0]
  const apiUrl = `https://api.github.com/repos/${form.value.owner}/${form.value.repo}/contents/${form.value.path}/${file?.name}`
  // console.log(apiUrl)
  const response = await fetch(apiUrl, {
    method: 'PUT',
    headers: {
      Authorization: `token ${form.value.pat}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      message: 'Upload image',
      content: form.value.img, // 纯 Base64 数据
    }),
  })

  if (!response.ok) {
    const errorData = await response.json()
    throw new Error(`GitHub API 错误: ${errorData.message || response.statusText}`)
  }

  const data = await response.json()
  console.log(data.content.html_url)
  console.log(data.content.download_url)
}
</script>
