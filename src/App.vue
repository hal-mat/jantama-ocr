<script setup lang="ts">
  import { Jimp } from "jimp";
  import cv from "@techstark/opencv-js";

  // https://jimp-dev.github.io/jimp/guides/browser/
  async function inputCheck(file: File, outputCanvasId: string): Promise<void> {
    console.log("called inputCheck");
    // inputからファイルを読み込む
    const fileArrayBuffer: ArrayBuffer = await file.arrayBuffer();
    // Jimpで読み込み
    const readImage = await Jimp.read(fileArrayBuffer);
    // canvasに描画
    const inputImageData: ImageData = new ImageData(
      Uint8ClampedArray.from(readImage.bitmap.data),
      readImage.bitmap.width,
      readImage.bitmap.height
    );
    const outputCanvas: HTMLCanvasElement = document.getElementById(outputCanvasId) as HTMLCanvasElement;
    const ctx: CanvasRenderingContext2D = outputCanvas.getContext("2d") as CanvasRenderingContext2D;
    ctx.putImageData(inputImageData, 0, 0);
  }
  // https://tekrog.com/jimp-with-typescript-on-browser

  // https://qiita.com/xenepic/items/4aa94bece50364002497
  let uploadedImage: File | null = null;

  async function handleFileUpload(e: Event): Promise<void> {
    console.log("called handleFileUpload");
    // ここでnullの危険性を消す
    const e_target = e.target as EventTarget & { files: FileList };
    const e_target_files: FileList = e_target.files;

    if (!e_target_files || e_target_files.length == 0) {
      return;
    }

    uploadedImage = e_target_files[0] as File; // -1不可
    console.log(typeof uploadedImage);
    // https://qiita.com/mmorino/items/6e6ed34bfb874c80a7f2
    // https://jimp-dev.github.io/jimp/guides/browser/
    inputCheck(uploadedImage, "input-check");
  }
</script>

<template>
  <h1>有効牌算出機</h1>
  <div>
    <a
      href="https://vite.dev"
      target="_blank"
    >
      <img
        src="/vite.svg"
        class="logo"
        alt="Vite logo"
      />
    </a>
    <a
      href="https://vuejs.org/"
      target="_blank"
    >
      <img
        src="./assets/vue.svg"
        class="logo vue"
        alt="Vue logo"
      />
    </a>
  </div>
  <!-- <HelloWorld msg="Vite + Vue" /> -->

  <div>
    <input
      type="file"
      id="file"
      accept="image/*"
      @change="handleFileUpload"
    />
    <!-- @change="onImageChange" -->
  </div>
  <canvas id="input-check"></canvas>
  <canvas id="gray"></canvas>
  <!-- https://qiita.com/dojyorin/items/00c167d0b4b41872b4a2 -->
</template>

<style scoped>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.vue:hover {
    filter: drop-shadow(0 0 2em #42b883aa);
  }
</style>
