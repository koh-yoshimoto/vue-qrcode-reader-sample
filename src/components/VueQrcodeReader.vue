<template>
  <div>
    <div>Please scan your QR codes...</div>
    <QrcodeStream @detect="onDetect"></QrcodeStream>
		<div class="results">
			<div class="results-title">Detected QR codes</div>
			<ul>
				<li v-for="(code, index) in codes" :key="index">
					{{ index + 1 }}: <a :href="code">{{ code }}</a>
				</li>
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
    const codes = ref([]);

    console.log("setup...");

    const onDetect = (result) => {
      const addedQrCode = result[result.length - 1];
      console.log("detected...", addedQrCode.rawValue);
      if (!codes.value.includes(addedQrCode.rawValue)) {
        codes.value.push(addedQrCode.rawValue);
      }
    };

    return {
      codes,
      onDetect,
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

.results-title {
	font-size: 1.5em;
}

.results {
	margin-top: 4px;
	outline: 2px solid #c2e9a3;
	border-radius: 2rem;
}
</style>
