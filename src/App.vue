<template>

  <div class="wrapper">
    <div class="wrapper__camera">
      <div v-show="!isCameraOpen" class="wrapper__camera__closed">
        <div v-if="permissionDenied" class="permission-denied">
          <img src="https://via.placeholder.com/50" alt="" />
          <h4>Permission Denied</h4>
          <p>
            KYC requires access to your camera and microphone. Please allow access and refresh your
            browser.
          </p>
        </div>
        <p v-if="!permissionDenied">
          {{
            !isManualUpload
              ? "Put your ID inside this area"
              : "Your uploaded photo will be showed here"
          }}
        </p>
        <!--HANDLER FOR OPEN BROWSER CAMERA-->
        <button
          v-if="!permissionDenied && !isNativeCamera && !isManualUpload"
          @click="startCamera"
        >Start Camera</button>
        <!--HANDLER FOR OPEN NATIVE CAMERA-->
        <label v-if="isNativeCamera" for="cameraFileInput">
          <span class="btn">
            <img src="https://via.placeholder.com/100" alt="" /> Start camera</span
          >
          <input
            id="cameraFileInput"
            ref="cameraFileInput"
            type="file"
            accept="image/*"
            capture="environment"
            @change="nativeCamera($event)"
          />
        </label>
        <!-- :disabled="successCapture || isLoadingOcr" -->
      </div>
      <div v-show="isCameraOpen" class="wrapper__camera__opened" :class="{ bordered: isEmpty }">
        <video v-show="isEmpty" ref="videoRef" autoplay playsinline></video>
        <canvas v-show="!isEmpty && !isNativeCamera && !isManualUpload" ref="canvasRef"></canvas>
        <!-- FIELD FOR NATIVE CAMERA -->
        <img v-show="isNativeCamera || isManualUpload" id="pictureFromCamera" />
        <!--ACTION-->
      </div>
      <div class="wrapper__camera__action">
        <div v-show="isCameraOpen || isManualUpload">File size: {{ fileSize }} MB</div>
        <div v-show="isCameraOpen || isManualUpload">Visibility: {{ visibilityStatus }}</div>

        <!-- HANDLER RETAKE or CAPTURE WITH BROWSER CAMERA -->
        <button
          v-show="isCameraOpen && !isNativeCamera && !isManualUpload"
          :disabled="isLoadingUpload"
          @click="isEmpty ? handleCapture() : startCamera()"
        >
        {{ isEmpty ? 'Capture' : isLoadingUpload ? '' : 'Retake' }}}
        </button>

        <!-- HANDLER MANUAL UPLOAD -->
        <label v-if="isManualUpload" for="cameraFileInput1">
          <span :class="successCapture || isLoadingUpload ? 'btn-disbaled upload' : 'btn upload'">
            <img src="https://via.placeholder.com/150" alt="" />Upload</span
          >
          <input
            id="cameraFileInput1"
            type="file"
            :disabled="successCapture || isLoadingUpload"
            accept="image/*"
            @change="nativeCamera($event)"
          />
        </label>

        <!-- HANDLER RETAKE WITH NATIVE CAMERA -->
        <label v-show="isCameraOpen && isNativeCamera" for="cameraFileInput">
          <span :class="successCapture || isLoadingUpload ? 'btn-disbaled' : 'btn'">
            <img src="https://via.placeholder.com/200" alt="" />Retake</span
          >
          <input
            id="cameraFileInput"
            :disabled="successCapture || isLoadingUpload"
            type="file"
            accept="image/*"
            capture="environment"
            @change="nativeCamera($event)"
          />
        </label>
      </div>
    </div>
  </div>

</template>

<script lang="ts" scoped>
import VxCamera from "vue-camera";
import { computed, defineComponent, ref } from "vue";
export default defineComponent({
  props: {
    isNativeCamera: {
      type: Boolean,
      default: false,
    },
    isLoadingUpload: {
      type: Boolean,
      default: false,
    },
    isManualUpload: {
      type: Boolean,
      default: false,
    },
    fileSize: {
      type: String,
      default: "0",
    },
    visibilityStatus: {
      type: String,
      default: "Good",
    },
  },
  setup() {
    const isCameraOpen = ref(false);
    const photoResult = ref();
    const videoRef = ref<HTMLVideoElement>();
    const canvasRef = ref<HTMLCanvasElement>();
    const cameraRef = ref();
    const isLoading = ref(false);
    const manualUploadIdentity = ref(false);
    const isEmpty = computed(() => photoResult.value == "");
    const permissionDenied = ref(false);
    const startCamera = async () => {
      // let checkPermission: NodeJS.Timer;
      const checkPermission = setInterval(() => {
        if (isCameraOpen.value) {
          clearInterval(checkPermission);
        } else {
          permissionDenied.value = true;
          clearInterval(checkPermission);
        }
      }, 1000);

      // eslint-disable-next-line @typescript-eslint/no-non-null-assertion
      cameraRef.value = await new VxCamera(videoRef.value!, canvasRef.value!)
        .setConstraint({
          video: {
            facingMode: "environment",
            height: {
              min: 480,
              max: 480,
              ideal: 480,
            },
            width: {
              min: 480,
              max: 480,
              ideal: 480,
            },
          },
          audio: false,
        })
        .requestPermission()
        .then((camera) => camera)
        .catch((err) => {
          console.error(err);
        });
      cameraRef.value?.start().finally(() => (isCameraOpen.value = true));
      photoResult.value = "";
    };

    const handleCapture = async () => {
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
      photoResult.value = await cameraRef.value?.snapAsBlob().then((data: any) => data);

      isLoading.value = true;
    };

    const successCapture = ref(false);
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    const nativeCamera = (e: any) => {
      const pictureFromCamera = document.getElementById("pictureFromCamera");
      isCameraOpen.value = true;
      const userInput = e.target.files[0];
      pictureFromCamera?.setAttribute("src", window.URL.createObjectURL(userInput));

      // const blob = new Blob([userInput], { type: "image/jpeg" });
      // successCapture.value = true;
    };
    return {
      permissionDenied,
      manualUploadIdentity,
      startCamera,
      isEmpty,
      isCameraOpen,
      videoRef,
      canvasRef,
      handleCapture,
      nativeCamera,
      successCapture,
    };
  },
});
</script>

<style lang="scss" scoped>

#cameraFileInput {
  display: none;
}
#cameraFileInput1 {
  display: none;
}

#pictureFromCamera {
  height: 300px;
  // width: 320px;
  object-fit: cover;
  border-radius: 8px;
}

#populatePhoto {
  height: auto;
  width: 320px;
  object-fit: cover;
  border-radius: 8px;
}

.btn {
  display: flex;
  align-items: center;
  gap: 8px;
  background-color: blue;
  color: white;
  padding: 8px 10px;
  border-radius: 8px;
  font-size: 14px;
  cursor: pointer;
  &.upload {
    :deep(img) {
      filter: invert(100%) sepia(100%) saturate(2%) hue-rotate(105deg) brightness(103%)
        contrast(101%);
    }
  }
}
.btn-disbaled {
  display: flex;
  align-items: center;
  gap: 8px;
  background-color: yellow;
  color: white;
  padding: 8px 10px;
  border-radius: 8px;
  font-size: 14px;
  cursor: not-allowed;
  &.upload {
    :deep(img) {
      filter: invert(100%) sepia(100%) saturate(2%) hue-rotate(105deg) brightness(103%)
        contrast(101%);
    }
  }
}

.wrapper {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  align-items: center;
  max-width: 100%;

  &__camera {
    max-width: 450px;
    width: 100%;
    &__closed {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 1rem;
      height: 300px;
      // width: 418px;
      padding: 0.75rem;
      border: 1px dashed red;
      border-radius: 8px;
      p {
        color: blue;
        font-family: "Inter", sans-serif;
        font-size: 1.4rem;
        font-weight: 600;
        letter-spacing: 0.2px;
      }
    }
    &__opened {
      display: flex;
      flex-direction: column;
      border-radius: 8px;
      position: relative;
      // &.bordered {
      //   padding: 0.75rem;
      // }
      video {
        height: 300px;
        object-fit: cover;
        border-radius: 4px;
        margin-bottom: 0;
        border: 1px dashed red;
      }
      canvas {
        height: 300px;
        object-fit: cover;
        border-radius: 4px;
        border: 1px dashed red;
      }
      :deep(.flip) {
        position: absolute;
        bottom: 1.5rem;
        right: 1.5rem;
        img {
          width: 20px;
          height: 20px;
        }
      }
    }
    &__action {
      display: flex;
      flex-direction: row;
      justify-content: space-between;
      align-items: center;
      font-size: 1.2rem;
      gap: 2rem;
      width: 100%;
      margin-top: 1rem;
      button:first-child {
        width: max-content;
      }
      :deep(.continue) {
        width: 50%;
        display: flex;
        justify-content: center;
        gap: 2rem;
        img {
          height: 14px;
          width: 16px;
        }
      }
    }
  }
}
.permission-denied {
  margin-bottom: 2rem;
  height: 300px;
  width: 418px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 4rem;
  h4 {
    margin-top: 0.5rem;
  }
  p {
    margin-top: 1rem;
    font-family: "Inter", sans-serif;
    font-style: normal;
    font-weight: 400;
    font-size: 14px;
    line-height: 20px;
    text-align: center;
    letter-spacing: 0.2px;
    color: #40444d;
  }
}
</style>
