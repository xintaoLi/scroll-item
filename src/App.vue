<template>
  <div id="app" style="width: 320px">
    <ScrollItem :list="items" :imgContainer="imgContainer">
      <template #pre>
        {{ "<" }}
      </template>
      <template #next>
        {{ ">" }}
      </template>
      <template #child="{ item }">
        <span style="width: 100px; font-size: 12px">{{ item.label }}</span>
      </template>
      <template #img>
        <span style="padding: 0 12px; border: solid 1px #ddd">img</span>
      </template>
    </ScrollItem>
    <div>Animate.css</div>
  </div>
</template>

<script>
import ScrollItem from "./components/scroll-item.vue";
export default {
  name: "App",
  components: {
    ScrollItem,
  },
  data() {
    return {
      imgContainer: {
        width: 52,
      },
      items: [
        { label: "全部", id: "all" },
        { label: "课程", id: "all2" },
        { label: "问答", id: "all3" },
        { label: "全部2", id: "all4" },
        { label: "全部3", id: "all5" },
      ],
    };
  },
  computed: {
    
  },
  methods: {

  }
};
</script>

<style>
@keyframes zoomOutUp {
  40% {
    opacity: 1;
    -webkit-transform: scale3d(0.475, 0.475, 0.475) translate3d(0, 60px, 0);
    transform: scale3d(0.475, 0.475, 0.475) translate3d(0, 60px, 0);
    -webkit-animation-timing-function: cubic-bezier(0.55, 0.055, 0.675, 0.19);
    animation-timing-function: cubic-bezier(0.55, 0.055, 0.675, 0.19);
  }
  to {
    opacity: 0;
    -webkit-transform: scale3d(0.1, 0.1, 0.1) translate3d(0, -2000px, 0);
    transform: scale3d(0.1, 0.1, 0.1) translate3d(0, -2000px, 0);
    -webkit-animation-timing-function: cubic-bezier(0.175, 0.885, 0.32, 1);
    animation-timing-function: cubic-bezier(0.175, 0.885, 0.32, 1);
  }
}
.animate__zoomOutUp {
  -webkit-animation-name: zoomOutUp;
  animation-name: zoomOutUp;
  -webkit-transform-origin: center bottom;
  transform-origin: center bottom;
}
</style>
