<script setup>
import InfoCard from "@/components/InfoCard.vue";
import { ref } from "vue";

import { initializeApp } from "firebase/app";
import { getFirestore, collection, getDocs, addDoc } from "firebase/firestore";

const isLoading = ref(true);
const db = getFirestore(initializeApp);
const docRef = collection(db, "posts");
const posts = ref([]);
const getCollections = (docRef) => {
  return new Promise((resolve, reject) => {
    getDocs(docRef)
      .then((collections) => {
        collections.docs.forEach((val) => {
          posts.value.push(val.data());
        });
        isLoading.value = false;
        resolve(collections.docs);
      })
      .catch((err) => {
        isLoading.value = true;
        alert(err.message);
      }),
      reject;
  });
};
getCollections(docRef);

const isPost = ref(false);

const content = ref("");

const [isBtnLoad, btnMessage] = [ref(false), ref("โพสต์")];

async function saveData() {
  [isBtnLoad.value, btnMessage.value] = [true, "กำลังโพสต์...."];
  const docRef = {
    content: content.value
  };
  return addDoc(collection(db, "posts"), docRef)
    .then(() => {
      window.location.reload();
    })
    .catch(() => {
      [isBtnLoad.value, btnMessage.value] = [false, "โพสต์"];
      alert("Failed to save");
    });
}
</script>

<template>
  <main>
    <div class="container">
      <img class="web-banner" :src="banner" alt="" />

      <div class="my-3">
        <div class="row">
          <div class="col-md-9 mx-auto">
            <h4 v-if="isLoading">กำลังโหลด...</h4>
            <div v-else>
              <button class="btn btn-primary" type="button" v-if="!isPost" @click="isPost = true">สร้างโพส</button>
              <div class="card mb-5" v-if="isPost">
                <div class="card-body">
                  <div class="row g-3">
                    <div class="col-md-12">
                      <div class="form-group">
                        <label class="mb-2" for="isIncome">เพิ่มโพสต์ใหม่</label>
                        <textarea
                          v-model="content"
                          rows="5"
                          class="form-control"
                          placeholder="เขียนอะไรสักหน่อย.."
                        ></textarea>
                      </div>

                      <button
                        class="btn mt-3 btn-primary w-100"
                        type="button"
                        @click="saveData"
                        :disabled="isBtnLoad"
                      >
                        {{ btnMessage }}
                      </button>
                    </div>
                  </div>
                </div>
              </div>

              <hr />

              <h4>โพสต์ทั้งหมด</h4>
              <InfoCard :content="stmt.content" v-for="(stmt, i) in posts" :key="i" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
.web-banner {
  width: 100%;
}
</style>
