<template>
  <v-main>
    <v-container>
      <div class="d-flex ga-4 align-center">
        <v-btn
          variant="plain"
          icon="mdi-arrow-left"
          color="primary"
          @click="router.push('/')"
        ></v-btn>
        <h2>{{ lotto.name }}</h2>
      </div>

      <p class="text-caption my-4">อัพเดท : {{ lotto.updated_at }}</p>

      <v-tabs v-model="tab" color="primary">
        <v-tab value="up">2 บน</v-tab>
        <v-tab value="down">2 ตัวล่าง</v-tab>
        <v-tab value="upDayOfWeek">รายวัน 2 บน</v-tab>
        <v-tab value="downDayOfWeek">รายวัน 2 ล่าง</v-tab>
      </v-tabs>

      <v-tabs-window v-model="tab">
        <v-tabs-window-item
          v-for="tabName in allTabs"
          :key="tabName"
          :value="tabName"
        >
          <div class="mt-4" v-if="tabName == 'up' || tabName == 'down'">
            <v-card v-for="item in lotto[tabName]">
              <v-card-title class="px-0 text-subtitle-1">
                <span v-if="item.count == 0">ไม่เคยออก</span>
                <span v-else>ออก {{ item.count }} ครั้ง</span>
              </v-card-title>
              <v-card-text class="px-0 text-subtitle-2 text-grey-darken-1">{{ item.numbers.join(', ') }}</v-card-text>
            </v-card>
          </div>

          <div v-else>
            <v-tabs v-model="dayTab" color="secondary">
              <v-tab v-for="(day, i) in dayOfWeeks" :value="i">{{ day }}</v-tab>
            </v-tabs>

            <v-tabs-window v-model="dayTab">
              <v-tabs-window-item v-for="item in lotto[tabName]" class="px-0">
                <v-table density="compact" class="py-4">
                  <thead>
                    <tr>
                      <th class="text-center">เลข</th>
                      <th class="text-center">จำนวนครั้ง</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="number in item.numbers">
                      <td class="text-center">{{ number[0] }}</td>
                      <td class="text-center">{{ number[1] }}</td>
                    </tr>
                  </tbody>
                </v-table>
              </v-tabs-window-item>
            </v-tabs-window>
          </div>
        </v-tabs-window-item>
      </v-tabs-window>
    </v-container>
  </v-main>
</template>

<script setup>
const config = useRuntimeConfig();
const route = useRoute();
const router = useRouter();
const slug = route.params.id;

const allTabs = ["up", "down", "upDayOfWeek", "downDayOfWeek"];
const dayOfWeeks = [
  "จันทร์",
  "อังคาร",
  "พุธ",
  "พฤหัสบดี",
  "ศุกร์",
  "เสาร์",
  "อาทิตย์",
];
const tab = ref();
const dayTab = ref();
const lotto = ref({
  name: "",
  up: [],
  down: [],
  upDayOfWeek: [],
  downDayOfWeek: [],
  updated_at: "",
});
const lottoStateKey = `lotto-${slug}`;
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

      lottoState.value.upDayOfWeek = makeDayOfWeek(data.up_day_of_week);
      lottoState.value.downDayOfWeek = makeDayOfWeek(data.down_day_of_week);
    } catch (error) {
      console.log(error);
    }
  }

  lotto.value = lottoState.value;
}

function makeDayOfWeek(data) {
  let response = [];
  data.forEach((item, i) => {
    response.push({
      day: dayOfWeeks[i],
      numbers: item,
    });
  });

  console.log(response);
  return response;
}
</script>
