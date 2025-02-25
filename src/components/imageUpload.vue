<template>
  <div class="container">
    <img class="image" src="../assets/image.jpg" alt="Current Image">

    <!-- 控制上传功能显示的按钮 -->
    <div v-if="uploadVisible" class="upload-container">

    </div>

    <!-- 浮动图标 -->
    <div class="toggle-icon" @click="toggleUploadVisibility">
      <img src="../assets/logo.gif" alt="Toggle Upload">
    </div>
    <van-popup v-model:show="uploadVisible" class="popup">
      <van-uploader v-model="fileList" :max-count="1" :after-read="afterRead">
      </van-uploader>
      <van-field v-model="token" label="密钥" placeholder="请输入密钥" />
      <van-button plain type="primary" @click="uploadImage" class="upload-btn">上传并替换图片</van-button>
    </van-popup>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { showToast } from 'vant';

const selectedFile = ref(null);
const fileList = ref([]);
const message = ref('');
const uploadVisible = ref(false); // 控制上传功能的显示和隐藏
const token = ref('')

function afterRead(file) {
  selectedFile.value = file.file
}
const handleFileUpload = (event) => {
  selectedFile.value = event.target.files[0];
};

const toggleUploadVisibility = () => {
  uploadVisible.value = !uploadVisible.value;
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
    const path = `src/assets/image.jpg`;

    try {
      const response = await updateGitHubFile(path, content);
      // 使用jsdelivr时，确保清空缓存并刷新
      const url = `https://cdn.jsdelivr.net/gh/Lyuan123/page-deploy@main/src/assets/image.jpg?${new Date().getTime()}`;
      message.value = '图片上传并成功替换！';
      uploadVisible.value = false
      showToast({
        message: '替换成功，请刷新该页面',
        position: 'top',
      });
    } catch (error) {
      // console.error(error);
      message.value = '图片上传失败，请重试。';
    }
  };
  reader.readAsDataURL(selectedFile.value);
};

const getFileSHA = async (path) => {
  const response = await fetch(
    `https://api.github.com/repos/Lyuan123/page-deploy/contents/${path}`,
    { headers: { Authorization: `Bearer ${token.value}` } }
  );

  if (!response.ok) return null; // 文件不存在时返回null
  const data = await response.json();
  return data.sha;
};

const updateGitHubFile = async (path, content) => {
  const url = `https://api.github.com/repos/Lyuan123/page-deploy/contents/${path}`;
  const existingSHA = await getFileSHA(path);
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), 50000); // 设置超时时间为5000毫秒（即5秒）
  const response = await fetch(url, {
    method: 'PUT',
    headers: {
      'Authorization': `Bearer ${token.value}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      message: '上传新图片',
      content: content,
      branch: 'main', // 你的分支名
      sha: existingSHA // 动态注入SHA值
    }),
    signal: controller.signal, // 传递 AbortSignal
  });

  if (!response.ok) {
    throw new Error('Failed to update file on GitHub.');
  }

  return response.json();
};
</script>

<style scoped>
/* 设置页面的布局 */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  overflow: hidden;
  background-color: #f0f0f0;
  /* 背景色 */
}

/* 设置图片样式 */
.image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.popup {
  padding: 10px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.upload-btn {
  margin: 10px 0px;
}

/* 上传区域 */
.upload-container {
  position: fixed;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  background-color: #fff;
  border: 2px dashed #00aaff;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
  max-width: 500px;
}

.upload-container input {
  margin-bottom: 10px;
  padding: 5px;
  border-radius: 4px;
  border: 1px solid #ccc;
  background-color: #f7f7f7;
}

.upload-container button {
  padding: 10px 20px;
  border: none;
  background-color: #00aaff;
  color: white;
  font-size: 16px;
  border-radius: 4px;
  cursor: pointer;
}

.upload-container button:hover {
  background-color: #0088cc;
}

/* 浮动图标 */
.toggle-icon {
  position: fixed;
  right: 20px;
  bottom: 20px;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.toggle-icon img {
  width: 30px;
  height: 30px;
  object-fit: cover;
}

/* 提示信息 */
.message {
  color: #333;
  font-size: 14px;
  margin-top: 10px;
  text-align: center;
}
</style>
