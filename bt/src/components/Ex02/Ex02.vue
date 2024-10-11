<template>
  <div>
    <h3>Bài 2:</h3>
    
  </div>
</template>

<script setup>
import axios from "axios";
import { onMounted } from "vue";
const getAllStudent = async () => {
  const res = await axios
    .get("http://localhost:8080/student")
    .then((res) => {
      console.log("user:", res.data);
    })
    .catch((err) => {
      console.log(err);
    });
    onMounted(()=>{
        getAllStudent();
    })
};
</script>

<style>
</style>