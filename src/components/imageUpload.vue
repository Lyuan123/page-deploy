<template>
  <div class="container">
    <img class="image" src="../assets/image.jpg" alt="Current Image">
    <div v-if="!isSuccess" class="toggle-icon" @dblclick="toggleUploadVisibility">
      <img src="../assets/logo.gif" alt="Toggle Upload">
    </div>
    <van-popup v-model:show="uploadVisible" class="popup" position="bottom">
      <van-uploader v-model="fileList" :max-count="1">
      </van-uploader>
      <van-field v-model="token" label="密钥" placeholder="请输入密钥" />
      <van-button plain type="primary" @click="uploadImage" class="upload-btn">上传并替换图片</van-button>
    </van-popup>
    <div class="loading-overlay" v-if="loading">
      <van-loading color="#1989fa" size="24px" vertical>
        <template #icon>
          <van-icon name="star-o" size="30" />
        </template>
        <span class="loading">加载中...</span>
      </van-loading>
    </div>
  </div>
</template>

<script setup>
import { ref, unref } from 'vue';
import { showToast } from 'vant';

const token = ref('')
const message = ref('');
const fileList = ref([]);
const loading = ref(false);
const isSuccess = ref(false);
const uploadVisible = ref(false);

const toggleUploadVisibility = () => {
  uploadVisible.value = !uploadVisible.value;
};

const uploadImage = async () => {
  if (!unref(fileList).length) {
    showToast({
      message: '请小可爱上传最少一张图片哦！',
      className: 'toast-message',
    })
    return;
  }

  if (!token.value) {
    showToast({
      message: '请小可爱输入密钥哦！',
      className: 'toast-message',
    })
    return;
  }

  const reader = new FileReader();
  reader.onload = async (e) => {
    const content = e.target.result.split(',')[1]; // 获取 Base64 编码的内容
    const path = `src/assets/image.jpg`;

    try {
      loading.value = !loading.value;
      const response = await updateGitHubFile(path, content);
      uploadVisible.value = false
      isSuccess.value = !isSuccess.value
      showToast({
        message: `替换成功，请2-3分钟后再次访问`,
        className: 'toast-message',
      })
    } catch (error) {
      showToast({
        message: `图片上传失败，原因:${error}`,
        className: 'toast-message',
      })
    } finally {
      loading.value = !loading.value;
    }
  };
  reader.readAsDataURL(fileList.value[0].file);
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
  const timeoutId = setTimeout(() => controller.abort(), 50000); // 设置超时时间5min
  const response = await fetch(url, {
    method: 'PUT',
    headers: {
      'Authorization': `Bearer ${token.value}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      message: '上传新图片',
      content: content,
      branch: 'main',
      sha: existingSHA
    }),
    signal: controller.signal, // 传递 AbortSignal
  });

  if (!response.ok) {
    throw new Error('Failed to update file on GitHub.');
  }

  return response.json();
};
</script>

<style>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  overflow: hidden;
}

/* 设置图片样式 */
.image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.popup {
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.upload-btn {
  margin: 10px 0px;
}

.toast-message {
  padding: 20px !important;
  font-weight: bold !important;
  font-size: 24px !important;
  background-color: black !important;
}

.loading {
  font-size: 24px;
  font-weight: 600;
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

.loading-overlay {
  position: fixed;
  /* 固定定位，覆盖整个页面 */
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  /* 水平居中 */
  align-items: center;
  /* 垂直居中 */
  background-color: rgba(255, 255, 255, 0.8);
  /* 半透明背景 */
  backdrop-filter: blur(5px);
  /* 背景虚化 */
  z-index: 9999;
  /* 确保在最上层 */
}

.loading {
  margin-top: 10px;
  /* 调整文字与图标的间距 */
}
</style>
