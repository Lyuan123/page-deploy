<template>
  <div>
    <h1>Upload and Replace Image</h1>
    <img :src="imageUrl" alt="Current Image">
    <input type="file" @change="handleFileUpload" accept="image/*">
    <button @click="uploadImage">Upload and Replace</button>
    <p v-if="message">{{ message }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const imageUrl = ref('https://raw.githubusercontent.com/Lyuan123/page-deploy/main/src/assets/image.jpg');
const selectedFile = ref(null);
const message = ref('');

const handleFileUpload = (event) => {
  selectedFile.value = event.target.files[0];
};

const uploadImage = async () => {
  if (!selectedFile.value) {
    message.value = '请小可爱上传最少一张图片哦！';
    return;
  }

  const reader = new FileReader();
  reader.onload = async (e) => {
    const content = e.target.result.split(',')[1]; // 获取 Base64 编码的内容
    const fileName = `uploaded-image-${Date.now()}.jpg`;
    const path = `images/${fileName}`;

    try {
      const response = await updateGitHubFile(path, content);
      imageUrl.value = response.data.content.html_url;
      message.value = 'Image uploaded and replaced successfully!';
    } catch (error) {
      console.error(error);
      message.value = 'Failed to upload image.';
    }
  };
  reader.readAsDataURL(selectedFile.value);
};

const updateGitHubFile = async (path, content) => {
  const url = `https://api.github.com/repos/Lyuan123/page-deploy/main/src/assets/image.jpg`;
  const token = '';

  const response = await fetch(url, {
    method: 'PUT',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      message: 'Upload new image',
      content: content,
      branch: 'main', // 你的分支名
    }),
  });

  if (!response.ok) {
    throw new Error('Failed to update file on GitHub.');
  }

  return response.json();
};
</script>
