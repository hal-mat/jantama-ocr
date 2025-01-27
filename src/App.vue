<script setup lang="ts">
  import { Jimp } from "jimp";

  // https://jimp-dev.github.io/jimp/guides/browser/
  // async function inputCheck(file: File, outputCanvasId: string): Promise<void> {
  // console.log("called inputCheck");
  // inputからファイルを読み込む
  // const arrayBuffer = await file.arrayBuffer();
  // Jimpで読み込み
  // const image = await Jimp.read(file);
  // canvasに描画
  // const imageData = new ImageData(Uint8ClampedArray.from(image.bitmap.data), image.bitmap.width, image.bitmap.height);
  // const outputCanvas: HTMLCanvasElement = document.getElementById(outputCanvasId) as HTMLCanvasElement;
  // const ctx: CanvasRenderingContext2D = outputCanvas.getContext("2d") as CanvasRenderingContext2D;
  // ctx.putImageData(imageData, 0, 0);
  // }
  // https://tekrog.com/jimp-with-typescript-on-browser
  // function showGray(file: File, outputCanvasId: string): void {
  // const outputCanvas = document.getElementById(outputCanvasId) as HTMLCanvasElement;
  // const ctx = outputCanvas.getContext("2d");
  // const arrayBuffer = file.arrayBuffer();
  // const image = Jimp.read(Buffer.from(arrayBuffer));
  // image.resize(256, 256);
  // const imageData = new ImageData(Uint8ClampedArray.from(image.bitmap.data), image.bitmap.width, image.bitmap.height);
  // ctx.putImageData(imageData, 0, 0);
  // }

  // https://qiita.com/xenepic/items/4aa94bece50364002497
  let uploadedImage: File | null = null;

  async function handleFileUpload(e: Event): void {
    const target = e.target as HTMLInputElement;

    if (!target.files || target.files.length == 0) {
      return;
    }
    uploadedImage = target.files[0]; // -1不可
    // https://qiita.com/mmorino/items/6e6ed34bfb874c80a7f2
    let reader: FileReader = new FileReader();
    reader.onload = (target) => {
      createImage(target.result);
    };

    console.log(typeof uploadedImage);
    // reader.readAsDataURL(uploadedImage);
    // https://jimp-dev.github.io/jimp/guides/browser/
  }
  function createImage(file: File) {
    console.log("createImage");
    // 画像サイズ調整
    Jimp.read(file, function (err, image) {
      image.getBase64(Jimp.MIME_PNG, function (err, src) {
        uploadedImageBlob = base64ToBlob(src);
        previewUploadedSrc = src;
      });
    });
  }

  function base64ToBlob(base64) {
    console.log("base64ToBlob");
    const bin = atob(base64.replace(/^.*,/, ""));
    const buffer = new Uint8Array(bin.length);
    for (let i = 0; i < bin.length; i++) {
      buffer[i] = bin.charCodeAt(i);
    }
    return new Blob([buffer.buffer], {
      type: "image/png",
    });
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
