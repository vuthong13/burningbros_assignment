<template>
  <div class="container">
    Product search:
    <input
      type="text"
      name="search"
      v-model="searchTxt"
      @input="fetchProduct()"
    />
    <div class="product-list">
      <div class="product" v-for="(item, i) in productArr" :key="i">
        <div class="product-image">
          <img :src="item.images[0]" alt="" />
        </div>
        <div class="product-info">
          <p class="product-ttl">{{ item.title }}</p>
          <p class="product-price">{{ item.price }}$</p>
        </div>
      </div>
    </div>
    <div id="observer"></div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, reactive, computed } from "vue";
import axios from "axios";

export default defineComponent({
  name: "HelloWorld",
  setup() {
    let searchTxt = ref("");
    let productArr: {
      id: number;
      title: string;
      price: number;
      images: string[];
    }[] = reactive([]);
    let currentPage = ref(1);
    const skip = computed(() => {
      return (currentPage.value - 1) * 20;
    });
    let pause = false;
    const fetchProduct = () => {
      if (searchTxt.value.length) {
        currentPage.value = 1;
        axios
          .get(
            "https://dummyjson.com/products/search?q=" +
              searchTxt.value +
              "&limit=20&skip=" +
              skip.value +
              "&select=title,price,images"
          )
          .then((response) => {
            productArr.length = 0;
            productArr.push(...response.data.products);
          });
      } else {
        pause = true;
        axios
          .get(
            "https://dummyjson.com/products?limit=20&skip=" +
              skip.value +
              "&select=title,price,images"
          )
          .then((response) => {
            productArr.push(...response.data.products);
            pause = false;
          });
      }
    };

    fetchProduct();

    const isElementInViewport = (el) => {
      var rect = el.getBoundingClientRect();
      return (
        rect.top >= 0 &&
        rect.left >= 0 &&
        rect.bottom <=
          (window.innerHeight || document.documentElement.clientHeight) &&
        rect.right <=
          (window.innerWidth || document.documentElement.clientWidth)
      );
    };
    document.addEventListener("scroll", () => {
      if (isElementInViewport(document.getElementById("observer")))
        if (!pause) {
          pause = true;
          currentPage.value++;
          fetchProduct();
        }
    });

    return { productArr, searchTxt, fetchProduct };
  },
});
</script>

<style scoped lang="scss">
@import "@/assets/css/style.scss";
</style>
