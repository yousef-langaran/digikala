<script setup lang="ts">

const images = ref([])

function getImages(url) {
  const request = new XMLHttpRequest();
  request.open("GET", url);
  request.send();
  request.onload = () => {
    if (request.status === 200) {
      const response = request.responseText;
      const hasan = JSON.parse(response)
      hasan.data.products.forEach(i => {
        const request2 = new XMLHttpRequest();
        request2.open("GET", `https://api.digikala.com/v2/product/${i.id}/`);
        request2.send();
        request2.onload = () => {
          if (request2.status === 200) {
            const response2 = request2.responseText;
            const hasan2 = JSON.parse(response2)
            hasan2.data.product.images.list.forEach(j => {
              images.value.push(j.url)
            })
          }
        };
      })
    }
  };
}

const index = ref(0)
const api = ref('')

function getData() {
  index.value++
  getImages(`${api.value}${index.value}`)
}

</script>

<template>
  <div>
    <label>آدرس صفحه</label>
    <input v-model="api">
  </div>
  <div>
    <label>شماره صفحه</label>
    <input v-model="index">
  </div>
  <div class="test">
    <a v-for="image in images" :href="image" target="_blank">
      <img :src="image">
    </a>
  </div>
  <div style="width: 100px; height: 50px; background-color: gray" @click="getData">
    LoadData
  </div>
</template>

<style scoped>
.test img {
  width: 200px;
  height: 200px;
}

</style>