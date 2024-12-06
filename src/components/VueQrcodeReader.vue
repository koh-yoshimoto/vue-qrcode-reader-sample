<template>
  <div>
    <div>Please scan your QR codes...</div>
    <QrcodeStream @detect="onDetect" :formats="['qr_code']" :track="paintBoundingBox"></QrcodeStream>
		<div class="result">
			<div class="result-title">Detected QR codes</div>
			<ul>
				<li v-for="code,index in result" :key="index">{{index+1}}: {{ code.rawValue }}</li>
			</ul>
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

    console.log("setup...");

		const paintOutline = (detectedCodes, ctx) => {
			for (const detectedCode of detectedCodes) {
				const [firstPoint, ...otherPoints] = detectedCode.cornerPoints

				ctx.strokeStyle = 'red'

				ctx.beginPath()
				ctx.moveTo(firstPoint.x, firstPoint.y)
				for (const { x, y } of otherPoints) {
					ctx.lineTo(x, y)
				}
				ctx.lineTo(firstPoint.x, firstPoint.y)
				ctx.closePath()
				ctx.stroke()
			}
		}

		const paintBoundingBox = (detectedCodes, ctx) => {
			for (const detectedCode of detectedCodes) {
				const {
					boundingBox: { x, y, width, height }
				} = detectedCode

				ctx.lineWidth = 2
				ctx.strokeStyle = '#007bff'
				ctx.strokeRect(x, y, width, height)
			}
		}

    const onDetect = (detectedCodes) => {
      result.value = detectedCodes
    }

    return {
      result,
      onDetect,
      paintOutline,
			paintBoundingBox
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
  margin: 2px 0;
  font-size:12px;
	overflow: hidden;
  text-overflow: ellipsis;
	text-decoration: underline;
}

.result-title {
	font-size: 1.5em;
}

.result {
	margin-top: 4px;
	outline: 2px solid #c2e9a3;
	border-radius: 2rem;
	width: 100%;
}
</style>
