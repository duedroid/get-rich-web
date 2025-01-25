<template>
  <v-main>
    <v-container>
      <div class="d-flex ga-4 align-center">
        <v-btn
          density="compact"
          icon="mdi-arrow-left"
          color="primary"
          @click="router.push('/')"
        ></v-btn>
        <h2>{{ lotto.name }}</h2>
      </div>
      <v-tabs v-model="tab" color="primary">
        <v-tab value="up">2 ตัวบน</v-tab>
        <v-tab value="down">2 ตัวล่าง</v-tab>
      </v-tabs>

      <v-tabs-window v-model="tab">
        <v-tabs-window-item
          v-for="tabName in allTabs"
          :key="tabName"
          :value="tabName"
        >
          <v-list lines="two">
            <v-list-item
              class="px-0"
              v-for="item in lotto[tabName]"
              :title="`ออก ${item.count} ครั้ง`"
              :subtitle="item.numbers.join(', ')"
            ></v-list-item>
          </v-list>
        </v-tabs-window-item>
      </v-tabs-window>

      <p class="text-caption">อัพเดท : {{ lotto.updated_at }}</p>
    </v-container>
  </v-main>
</template>

<script setup>
const route = useRoute();
const router = useRouter();
const slug = route.params.id;

const allTabs = ["up", "down"];
const tab = ref();
const lotto = ref({
  name: "",
  up: [],
  down: [],
  updated_at: "",
});
const lottoStateKey = `lotto-${slug}`
const lottoState = useState(lottoStateKey, () => null);

onMounted(async () => {
  await getLotto();
});

async function getLotto() {
  if (lottoState.value === null) {
    try {
      const response = await fetch(
        `${config.public.apiBase}/api/v1/lotto/${slug}`,
        {
          method: "GET",
        }
      );

      const data = await response.json();
      lottoState.value = data;
    } catch (error) {
      console.log(error);
    }
  }

  lotto.value = lottoState.value;
}
</script>
