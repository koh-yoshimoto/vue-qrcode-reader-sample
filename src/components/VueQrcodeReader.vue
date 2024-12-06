<template>
  <div>
    <select v-model="selectedConstraints">
      <option
        v-for="option in constraintOptions"
        :key="option.label"
        :value="option.constraints"
      >
        {{ option.label }}
      </option>
    </select>

    <div>Please scan your QR codes...</div>
    <QrcodeStream
      :paused="paused"
      :constraints="selectedConstraints"
      @detect="onDetect"
      :formats="['qr_code']"
      :track="paintBoundingBox"
      @camera-on="onCameraReady"
    ></QrcodeStream>

    <div class="result">
      <div class="result-title">Detected QR codes</div>
      <ul>
        <li v-for="(code, index) in result" :key="index">
          [{{ code.cornerPoint.x }}]: {{ code.rawValue }}
        </li>
      </ul>
    </div>
    <div class="result">
    <div class="result-title">Result</div>
			<div v-if="carType" class="result-title">
				Car Type: {{ carType }}
			</div>
			<div v-if="qr2" class="decoded-value">
				QR2: {{ qr2 }}
			</div>
			<div v-if="qr3" class="decoded-value">
				QR3: {{ qr3 }}
			</div>
  </div>
    </div>
</template>

<script>
import { defineComponent, ref } from "vue";
import { QrcodeStream } from "vue-qrcode-reader";

export default defineComponent({
  name: "MultiQrCodeReader",
  components: {
    QrcodeStream,
  },
  setup() {
    const result = ref([]);
    const position = ref([]);
    const paused = ref(false);
    const qr2 = ref("");
		const qr3 = ref("");
    const carType = ref("");

    const selectedConstraints = ref({ facingMode: "environment" });
    const defaultConstraintOptions = [
      { label: "rear camera", constraints: { facingMode: "environment" } },
      { label: "front camera", constraints: { facingMode: "user" } },
    ];
    const constraintOptions = ref(defaultConstraintOptions);

    console.log("setup...");

    const onCameraReady = async () => {
      // NOTE: on iOS we can't invoke `enumerateDevices` before the user has given
      // camera access permission. `QrcodeStream` internally takes care of
      // requesting the permissions. The `camera-on` event should guarantee that this
      // has happened.
      const devices = await navigator.mediaDevices.enumerateDevices();
      const videoDevices = devices.filter(({ kind }) => kind === "videoinput");

      constraintOptions.value = [
        ...defaultConstraintOptions,
        ...videoDevices.map(({ deviceId, label }) => ({
          label: `${label} (ID: ${deviceId})`,
          constraints: { deviceId },
        })),
      ];
    };

    const paintOutline = (detectedCodes, ctx) => {
      for (const detectedCode of detectedCodes) {
        const [firstPoint, ...otherPoints] = detectedCode.cornerPoints;

        ctx.strokeStyle = "red";

        ctx.beginPath();
        ctx.moveTo(firstPoint.x, firstPoint.y);
        for (const { x, y } of otherPoints) {
          ctx.lineTo(x, y);
        }
        ctx.lineTo(firstPoint.x, firstPoint.y);
        ctx.closePath();
        ctx.stroke();
      }
    };

    const paintBoundingBox = (detectedCodes, ctx) => {
      for (const detectedCode of detectedCodes) {
        const {
          boundingBox: { x, y, width, height },
        } = detectedCode;

        ctx.lineWidth = 4;
        ctx.strokeStyle = "#007bff";
        ctx.strokeRect(x, y, width, height);
      }
    };

    const onDetect = (detectedCodes) => {
      //重複しないrawValueをresultに追加
      for (let i = 0; i < detectedCodes.length; i++) {
        const exists = result.value.some(
          (item) => item.rawValue === detectedCodes[i].rawValue,
        );
        if (!exists) {
          result.value.push({
            rawValue: detectedCodes[i].rawValue,
            cornerPoint: {
              x: Math.round(detectedCodes[i].cornerPoints[0].x),
              y: Math.round(detectedCodes[i].cornerPoints[0].y),
            }, });
          //resultをcornerPoints[0]のx座標で降順ソート
          result.value.sort((a, b) => {
            if (a.cornerPoint.x < b.cornerPoint.x) {
              return 1;
            } else {
              return -1;
            }
          });
          console.log(result.value);
          if (result.value.length == 5) {
            paused.value = true;
						qr2.value = result.value[1].rawValue + result.value[0].rawValue;
						qr3.value = result.value[4].rawValue + result.value[3].rawValue + result.value[2].rawValue;

          }
        }
      }
    };

    return {
      result,
      onDetect,
      paintOutline,
      paintBoundingBox,
      constraintOptions,
      onCameraReady,
      paused,
			carType,
			qr2, qr3,
    };
  },
});
</script>

<style scoped>
ul {
  list-style: none;
  padding: 0;
}

li {
  margin-bottom: 4px;
  font-size: 12px;
  overflow: hidden;
  text-overflow: ellipsis;
  text-decoration: underline;
}

.decoded-value {
	font-size: 1.0rem;
	margin: 4px 0;
	text-align: left;
}

.result {
  margin-top: 4px;
  outline: 2px solid #c2e9a3;
  border-radius: 1rem;
  width: 100%;
	padding: 4px;
}
</style>
