<script setup lang="ts">
  import { Jimp } from "jimp";
  import cv from "@techstark/opencv-js";

  // https://jimp-dev.github.io/jimp/guides/browser/
  async function inputCheck(file: File, outputCanvasId: string): Promise<void> {
    // inputからファイルを読み込む
    const fileArrayBuffer: ArrayBuffer = await file.arrayBuffer();
    // Jimpで読み込み
    const readImage = await Jimp.read(fileArrayBuffer);
    // 最大範囲を指定してリサイズ
    // readImage.scaleToFit({ h: 1280, w: 1280 });
    // readImage.resize({ h: 1280, w: 1280 });
    console.log([readImage.bitmap.width, readImage.bitmap.height]);
    // canvasに描画
    const inputImageData: ImageData = new ImageData(
      Uint8ClampedArray.from(readImage.bitmap.data),
      readImage.bitmap.width,
      readImage.bitmap.height
    );
    const outputCanvas: HTMLCanvasElement = document.getElementById(outputCanvasId) as HTMLCanvasElement;
    const ctx: CanvasRenderingContext2D = outputCanvas.getContext("2d", {
      willReadFrequency: true,
    }) as CanvasRenderingContext2D;
    outputCanvas.width = readImage.bitmap.width;
    outputCanvas.height = readImage.bitmap.height;
    ctx.putImageData(inputImageData, 0, 0);
    console.log("put inputCheck");
  }
  // https://tekrog.com/jimp-with-typescript-on-browser
  // async function monochrome(inputCanvasId: string, outputCanvasId: string): Promise<void> {
  function monochrome(inputCanvasId: string, outputCanvasId: string): void {
    console.log("called monochrome");
    let src = cv.imread(inputCanvasId);
    let gray_dst = new cv.Mat();
    cv.cvtColor(src, gray_dst, cv.COLOR_RGBA2GRAY, 0);
    src.delete();
    let monochrome_dst = new cv.Mat();
    cv.threshold(gray_dst, monochrome_dst, 177, 200, cv.THRESH_BINARY);
    // cv.threshold(gray_dst, monochrome_dst, 0.7, 1, cv.THRESH_BINARY);
    gray_dst.delete();
    cv.imshow(outputCanvasId, monochrome_dst);
    monochrome_dst.delete();
  }

  function fromImageToString(inputCanvasId: string): string {
    console.log("called monochrome");
    let src = cv.imread(inputCanvasId);
    let gray_dst = new cv.Mat();
    cv.cvtColor(src, gray_dst, cv.COLOR_RGBA2GRAY, 0);
    src.delete();
    let monochrome_dst = new cv.Mat();
    cv.threshold(gray_dst, monochrome_dst, 177, 200, cv.THRESH_BINARY);
    // cv.threshold(gray_dst, monochrome_dst, 0.7, 1, cv.THRESH_BINARY);
    gray_dst.delete();
    monochrome_dst.convertTo(monochrome_dst, cv.CV_8UC1);
    let contours = new cv.MatVector();
    let hierarchy = new cv.Mat();
    // const paiList: string[] = ["1p", "2p", "3p", "4p", "5p", "6p", "7p", "8p", "9p", "1s", "2s", "3s", "4s", "5s", "6s", "7s", "8s", "9s", "1m", "2m", "3m", "4m", "5m", "6m", "7m", "8m", "9m", "東", "南", "西", "北", "白", "発", "中"];
    const paiList: string[] = [
      "1p",
      "2p",
      "3p",
      "4p",
      "5p",
      "6p",
      "7p",
      "8p",
      "9p",
      "1s",
      "2s",
      "3s",
      "4s",
      "5s",
      "6s",
      "7s",
      "8s",
      "9s",
      "1m",
      "2m",
      "3m",
      "4m",
      "5m",
      "6m",
      "7m",
      "8m",
      "9m",
      "1z",
      "2z",
      "3z",
      "4z",
      "5z",
      "6z",
      "7z",
    ];
    let answerArray: string[] = [];
    paiList.forEach((pai: string) => {
      let templ = cv.imread(pai);
      let dst = new cv.Mat();
      let mask = new cv.Mat();
      // https://gist.github.com/lucasvieceli/7e0e697861d6b49201491f135c03ebb4
      cv.matchTemplate(monochrome_dst, templ, dst, cv.TM_CCOEFF_NORMED);
      cv.findContours(monochrome_dst, contours, hierarchy, cv.RETR_EXTERNAL, cv.CHAIN_APPROX_SIMPLE);
      [...Array(contours.size())].map(() => answerArray.push(pai));
      templ.delete();
      dst.delete();
      mask.delete();
      if (answerArray.length == 14) {
        return;
      }
    });
    monochrome_dst.delete();
    return answerArray.join();
  }

  // https://qiita.com/xenepic/items/4aa94bece50364002497
  let uploadedImage: File | null = null;

  async function handleFileUpload(e: Event): Promise<void> {
    console.log("called handleFileUpload");
    // ここでnullの危険性を消す
    const eTarget = e.target as EventTarget & { files: FileList };
    const eTargetFiles: FileList = eTarget.files;

    if (!eTargetFiles || eTargetFiles.length == 0) {
      alert("ファイルが選択されていません");
      return;
    }

    uploadedImage = eTargetFiles[0] as File; // -1不可
    // https://qiita.com/mmorino/items/6e6ed34bfb874c80a7f2
    // https://jimp-dev.github.io/jimp/guides/browser/
    await inputCheck(uploadedImage, "input-check");
    monochrome("input-check", "monochrome");
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
  <canvas id="monochrome"></canvas>
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
  canvas {
    /* width: 1280px; */
    width: 100%;
    height: auto;
  }
</style>
