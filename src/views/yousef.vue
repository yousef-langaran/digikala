<script setup>
import axios from 'axios'
import {ref} from 'vue'
const images = ref([])
const index = ref(0)
const toIndex = ref(100)
const api = ref('')
const texts = ref([])
const toDataURL = async (url) =>
    await fetch(url)
        .then(response => response.blob())
        .then(blob => new Promise((resolve, reject) => {
          const reader = new FileReader()
          reader.onloadend = () => resolve(reader.result)
          reader.onerror = reject
          reader.readAsDataURL(blob)
        }))

function dataURLtoFile(dataurl, filename) {
  var arr = dataurl.split(','), mime = arr[0].match(/:(.*?);/)[1],
      bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
  while(n--){
    u8arr[n] = bstr.charCodeAt(n);
  }
  return new File([u8arr], filename, {type:mime});
}
async function imageToText(imageLink){
  var formData = new FormData();
  const regex = /^[a-zA-Z0-9]+$/;
  formData.set('image', imageLink)
  await axios.post('https://api.api-ninjas.com/v1/imagetotext',formData,
      {
        headers:{
          'X-Api-Key': 'vSwVB0PWremjsY+9JfH+rQ==V6WaMWwHSp5MVmdb'
        }
      }).then(res=>{
    res.data.forEach(e=>{
      if (regex.test(e.text) && e.text.length > 7 && e.text.length < 13){
        texts.value.push(e.text)
      }
    })
  })
}
async function getImages(url) {
  await axios.get(url).then(async (res)=>{
    await setData(res.data)
  })
}

async function setData(hasan){
  for (const i in hasan.data.products) {
    await axios.get(`https://api.digikala.com/v2/product/${hasan.data.products[i].id}/`).then(async (res) => {
      for (const j in res.data.data.product.images.list) {
        if (extractFilenameFromURL(res.data.data.product.images.main.url).split("_")[1]){
          if(extractFilenameFromURL(res.data.data.product.images.main.url).split("_")[1]?.substring(0,4) !== extractFilenameFromURL(res.data.data.product.images.list[j].url)?.split("_")[1]?.substring(0,4)){
            images.value.push(res.data.data.product.images.list[j].url)
            await toDataURL(res.data.data.product.images.list[j].url).then(async (dataUrl) => {
              var fileData = dataURLtoFile(dataUrl, "imageName.jpg");
              // await imageToText(fileData)
            })
          }
        }
        else{
          images.value.push(res.data.data.product.images.list[j].url)
          console.log(extractFilenameFromURL(res.data.data.product.images.main.url))
        }

      }
    })
  }
}

function extractFilenameFromURL(url) {
  // جدا کردن بخش فایل از URL با استفاده از split('/')
  const parts = url.toString().split('/');
  // گرفتن آخرین بخش از آرایه برای بدست آوردن نام فایل
  const lastPart = parts[4];

  // جدا کردن پارامترهای URL برای گرفتن نام فایل بدون پارامترها
  const filename = lastPart.split('?')[0];
  return filename;
}

async function getData() {
  for (let i = index.value; i< toIndex.value; i++){
    await getImages(`${api.value}${i}`)
    index.value++
  }
}

</script>

<template>
  <a href="https://chromewebstore.google.com/detail/cors-unblock/lfhmikememgdcahcdlaciloancbhjino?pli=1" target="_blank">
    حتما این اکستنشن رو نصب و فعال کن
  </a>
  <div class="myForm">
  <div>
    <label>آدرس صفحه</label>
    <input v-model="api">
  </div>
  <div>
    <label>از صفحه</label>
    <input type="number" v-model="index">
  </div>
    <div>
      <label>تا صفحه</label>
      <input type="number" v-model="toIndex">
    </div>
  </div>
  <div class="myMain">
    <a v-for="image in images" :href="image" target="_blank">
      <img loading="lazy" :src="image">
    </a>
  </div>
  <div class="myButton" @click="getData">
    LoadData
  </div>
</template>

<style scoped>
.myMain img {
  width: 200px;
  height: 200px;
}
.myButton{
  width: 100%;
  height: 58px;
  border-radius: 16px;
  background-color: #5252ff;
  display: grid;
  place-items: center;
  font-size: 24px;
  font-weight: 800;
  color: white;
  cursor: pointer;
  margin-top: 16px;
}
.myForm{
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
  font-size: 24px;
  position: fixed;
  top: 32px;
}
input{
  height: 38px;
  font-size: 20px;
}
.myMain{
  margin-top: 100px;
}
</style>