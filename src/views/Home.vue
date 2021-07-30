<template>
  <div class="home p-4 w-full mx-auto">
    <div class="bg-white/40 p-5 h-56 rounded-lg shadow-lg">
      <h1 class="py-2 px-1 text-center text-xl font-bold mb-2">
        Ketersediaan Tempat Tidur Rumah Sakit berdasarkan Kota 🚀
      </h1>
      <div class="mb-4">
        <div class="inline-flex mr-4 rounded-lg">
          <input
            type="radio"
            name="room_type"
            id="roomCovid"
            value="1"
            checked
            hidden
            @change="changeJenis($event)"
          />
          <label
            for="roomCovid"
            class="
              radio
              text-center
              self-center
              py-2
              px-4
              rounded-lg
              cursor-pointer
              hover:opacity-75
            "
            >Covid</label
          >
        </div>
        <div class="inline-flex rounded-lg">
          <input
            type="radio"
            name="room_type"
            id="roomPublic"
            value="2"
            hidden
            @change="changeJenis($event)"
          />
          <label
            for="roomPublic"
            class="
              radio
              text-center
              self-center
              py-2
              px-4
              rounded-lg
              cursor-pointer
              hover:opacity-75
            "
            >Non-Covid</label
          >
        </div>
      </div>
      <input
        type="text"
        v-model="input"
        @input="changeInput"
        @keyup.enter="handleKeyPress"
        placeholder="Ketikkan Kota..."
        class="
          placeholder-gray-500
          rounded-sm
          h-10
          border-4 border-gray-500
          w-80
          mx-auto
          text-gray-900 text-center
          placeholder-transparent
          focus:outline-none
        "
      />
      <div v-if="filteredCity.length > 0">
        <ul>
          <li
            class="
              mx-auto
              w-80
              bg-green-300
              font-medium
              cursor-pointer
              hover:bg-green-500
              transition-all
            "
            v-for="prov in filteredCity"
            :key="prov"
            @click="getInfoBed(prov)"
          >
            {{ prov.name }}
          </li>
        </ul>
      </div>
    </div>
  </div>
  <div v-if="loading">
    <div
      class="
        border border-gray-400
        shadow
        rounded-md
        p-4
        max-w-sm
        w-full
        mx-auto
      "
    >
      <div class="animate-pulse flex space-x-4">
        <div class="rounded-full bg-gray-400 h-12 w-12"></div>
        <div class="flex-1 space-y-4 py-1">
          <div class="h-4 bg-gray-400 rounded w-3/4"></div>
          <div class="space-y-2">
            <div class="h-4 bg-gray-400 rounded"></div>
            <div class="h-4 bg-gray-400 rounded w-5/6"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div v-if="error">Terjadi Error</div>
  <div v-else-if="resultEnd.length > 0">
    <div
      v-for="data in resultEnd"
      :key="data"
      class="
        border border-gray-400
        shadow-lg
        rounded-md
        max-w-sm
        w-full
        mx-auto
        my-3
        p-4
        bg-gray-300
      "
    >
      <p>Rumah Sakit : {{ data.name }}</p>
      <p>Alamat : {{ data.address }}</p>
      <p>Tempat Tidur Yang Tersedia : {{ data.available_bed }}</p>
      <p>Antrian : {{ data.bed_queue }}</p>
      <!-- <p>Detail Bed : {{ data.bed_detail_link }}</p> -->
      <p>Hotline : {{ data.hotline || "Tidak tersedia" }}</p>
      <p>{{ data.updated_at_minutes }}</p>
    </div>
  </div>
  <button
    @click="scrollToTop"
    class="z-10 fixed bottom-5 right-5 p-3 bg-gray-300 rounded-full shadow-md"
  >
    TOP
  </button>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from "@/components/HelloWorld.vue";
import { apiBed } from "../utils/api";
import { ref, reactive, toRefs } from "@vue/reactivity";
import { cities } from "../utils/cities";
import { provinces } from "../utils/provinces";
import cheerio from "cheerio";
import { parseUrl } from "query-string";

export default {
  name: "Home",
  // components: {
  //   HelloWorld,
  // },
  setup() {
    const checkJenis = ref(1);
    const isScroll = ref(false);
    const input = ref("");
    const error = ref(false);
    const loading = ref(false);
    const result = reactive({
      filteredCity: [],
      resultProv: {},
      resultCity: {},
      resultEnd: [],
    });

    const changeInput = () => {
      if (input.value) {
        result.filteredCity = cities
          .filter((city) =>
            city.name.toLowerCase().includes(input.value.toLowerCase())
          )
          .slice(0, 5);
      } else {
        result.filteredCity = [];
      }
    };

    const handleKeyPress = () => {
      if (result.filteredCity.length > 0) {
        result.resultCity = result.filteredCity[0];
        getInfoBed(result.resultCity);
      }
    };

    const getInfoBed = async (e) => {
      result.resultEnd = [];
      result.resultProv = provinces.filter((item) => item.key == e.key)[0];
      result.resultCity = e;
      input.value = e.name;
      loading.value = true;
      try {
        result.filteredCity = [];
        const { data } = await apiBed.get(
          `rumah_sakit?jenis=${checkJenis.value}&propinsi=${result.resultProv.key}&kabkota=${result.resultCity.code}#`
        );
        const $ = cheerio.load(data);
        const hospitalDetail = [];
        const bedAvailable = [];
        const updatedAt = [];
        const bedQueue = [];
        const hotline = [];
        const bedDetailLink = [];
        if (checkJenis.value === 1) {
          $("div")
            .find(".col-md-5.text-right")
            .filter((i, el) => {
              const data = $(el);
              const getBedInfo = data
                .find(":nth-child(2)")
                .first()
                .text()
                .split(" ")
                .filter((i) => i !== "")
                .join(" ")
                .split("\n")[1]
                .trim()
                .split(" ");
              const getBedQueue = data
                .find(":nth-child(3)")
                .first()
                .text()
                .split(" ")
                .filter((i) => i !== "")
                .join(" ")
                .split("\n")[1]
                .trim()
                .split(" ");
              const getLastUpdatedAt = data
                .find(":nth-child(4)")
                .first()
                .text()
                .split(" ")
                .filter((i) => i !== "")
                .join(" ")
                .split("\n")[1]
                .trim()
                .split(" ");
              if (getBedInfo[0] === "Bed") {
                bedAvailable.push(0);
              } else if (getBedInfo[0] === "Tersedia") {
                bedAvailable.push(Number(getBedInfo[1]));
              }
              if (getBedQueue[0] === "tanpa") {
                bedQueue.push(0);
              } else if (getBedQueue[0] === "dengan") {
                bedQueue.push(Number(getBedQueue[2]));
              }
              if (getLastUpdatedAt[1] === "kurang") {
                updatedAt.push(0);
              } else if (getLastUpdatedAt[2] === "menit") {
                updatedAt.push(
                  getRelativeLastUpdatedTime(Number(getLastUpdatedAt[1]))
                );
              } else if (getLastUpdatedAt[2] === "jam") {
                updatedAt.push(
                  getRelativeLastUpdatedTime(Number(getLastUpdatedAt[1]) * 60)
                );
              }
            });
          // Address
          $("div")
            .find(".col-md-7")
            .filter((i, el) => {
              const data = $(el);
              if (
                data
                  .first()
                  .text()
                  .split(" ")
                  .filter((i) => i !== "")
                  .join(" ")
                  .split("\n")
              ) {
                const hospitalName = data.find("h5").first().text();
                const hospitalAddress = data
                  .first()
                  .text()
                  .split(" ")
                  .filter((i) => i !== "")
                  .join(" ")
                  .split("\n")[2]
                  .trim();
                hospitalDetail.push({
                  name: hospitalName,
                  address: hospitalAddress,
                });
              }
            });
          // Hotline
          $("div")
            .find(".card-footer")
            .find("span")
            .filter((i, el) => {
              const data = $(el);
              const hotlineSelector = data.first().text();
              const hotlineSelectorSplitted = hotlineSelector.split("/");
              if (hotlineSelector.includes("tidak tersedia")) {
                hotline.push("");
              } else if (hotlineSelectorSplitted.length > 1) {
                hotline.push(
                  hotlineSelectorSplitted.join(",").replace(/ /g, "")
                );
              } else {
                hotline.push(hotlineSelector.replace(/ /g, ""));
              }
            });
          $("div")
            .find(".card-footer")
            .find("a")
            .filter((i, el) => {
              const data = $(el);
              if (data.first().attr("href").includes("http")) {
                const href = data.first().attr("href");
                const { query } = parseUrl(href);
                bedDetailLink.push({
                  link: href,
                  hospital_code: query.kode_rs,
                });
              }
            });
          const hospitalArray = hospitalDetail.map((hos, idx) => ({
            name: hos.name,
            address: hos.address,
            available_bed: Number(bedAvailable[idx]) || 0,
            bed_queue: bedQueue[idx],
            hotline: hotline[idx],
            bed_detail_link: bedDetailLink[idx].link,
            hospital_code: bedDetailLink[idx].hospital_code,
            updated_at_minutes: updatedAt[idx],
          }));
          result.resultEnd = hospitalArray;
        } else {
          // get hospital Name and Address

          $("div")
            .find(".cardRS.col-md-12.mb-2")
            .filter((i, el) => {
              const bed = [];
              const updatedAt = [];
              const hotline = [];
              const data = $(el);
              const hospitalName = data
                .find(".col-md-5")
                .find("h5")
                .text()
                .trim();
              const hospitalAddress = data
                .find(".col-md-5")
                .find("p")
                .text()
                .trim();
              data.find(".col-md-4.text-center.mb-2").filter((i, el) => {
                const data = $(el);
                const getBedInfo = data
                  .find(".card-body")
                  .find(":nth-child(1)")
                  .text()
                  .trim();
                const getBedInfoClass = data
                  .find(".card-body")
                  .find(":nth-child(2)")
                  .text()
                  .trim();
                const getBedInfoAddress = data
                  .find(".card-body")
                  .find(":nth-child(3)")
                  .text()
                  .trim()
                  .replace(/Di Ruang/g, "")
                  .trim();
                const getLastUpdatedAt = data
                  .find(".card-footer")
                  .find(":nth-child(1)")
                  .text()
                  .trim();
                bed.push({
                  amount: +getBedInfo,
                  class: getBedInfoClass,
                  address: getBedInfoAddress,
                });
                updatedAt.push(getLastUpdatedAt);
              });

              hotline.push(
                data.find(".card-footer.text-right").find("span").text()
              );

              hospitalDetail.push({
                name: hospitalName,
                address: hospitalAddress,
                bed,
                updatedAt,
                hotline,
              });
            });
        }

        result.filteredCity = [];
      } catch (error) {
        error.value = true;
      } finally {
        loading.value = false;
        input.value = "";
      }
    };
    const changeJenis = (e) => {
      checkJenis.value = e.target.value;
    };

    const getRelativeLastUpdatedTime = (lastUpdatedInMinutes) => {
      const relativeTimeFormat = new Intl.RelativeTimeFormat("id");
      let displayedTime;

      if (lastUpdatedInMinutes >= 60) {
        displayedTime = relativeTimeFormat.format(
          -Math.round(lastUpdatedInMinutes / 60),
          "hour"
        );
      } else {
        if (-Math.round(lastUpdatedInMinutes) === 0) {
          displayedTime = "kurang dari 1 menit yang lalu";
        } else {
          displayedTime = relativeTimeFormat.format(
            -Math.round(lastUpdatedInMinutes),
            "minute"
          );
        }
      }

      return displayedTime;
    };

    window.onscroll = function () {
      scrollFunction();
    };

    function scrollFunction() {
      if (
        document.body.scrollTop > 20 ||
        document.documentElement.scrollTop > 20
      ) {
        isScroll.value = true;
      } else {
        isScroll.value = false;
      }
    }

    const scrollToTop = () => {
      document.body.scrollTop = 0;
      document.documentElement.scrollTop = 0;
    };

    return {
      isScroll,
      input,
      checkJenis,
      handleKeyPress,
      ...toRefs(result),
      error,
      loading,
      changeInput,
      getInfoBed,
      changeJenis,
      scrollToTop,
    };
  },
};
</script>
<style scoped>
html {
  scroll-behavior: smooth;
}
input:checked ~ .radio {
  color: white;
  background-color: rgb(67, 170, 67);
}
input ~ .radio {
  color: rgb(102, 100, 100);
  background-color: rgb(185, 184, 184);
}
</style>
