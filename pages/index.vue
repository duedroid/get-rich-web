<template>
  <v-main>
    <v-container class="d-flex flex-column ga-4">
      <div v-for="group in lottoList">
        <h2 class="mb-4">{{ lottoGroupMap[group.group] }}</h2>
        <v-row>
          <v-col cols="6" v-for="lotto in group.lotto_list">
            <v-card @click="router.push(`/lotto/${lotto.slug}`)">
              <v-card-text>
                <div class="d-flex ga-2">
                  <img :width="20" :src="`/img/${lotto.country}.svg`" />
                  <p>{{ lotto.name }}</p>
                </div>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </div>
    </v-container>
  </v-main>
</template>

<script setup>
const config = useRuntimeConfig()
const router = useRouter()

const lottoList = useState('lottoList', () => null)
const lottoGroupMap = {
  "laos": "หวยลาว",
  "vietnam": "หวยเวียดนาม",
  "stock": "หวยหุ้น",
  "stock-vip": "หวยหุ้น VIP",
  "thai-auth": "หวยทางการ",
};

onMounted(async () => {
  await getLotto();
});

async function getLotto() {
  if (lottoList.value !== null) {
    return
  }

  try {
    const response = await fetch(
      `${config.public.apiBase}/api/v1/lotto`,
      {
        method: "GET",
      }
    );

    const data = await response.json();

    lottoList.value = Object.values(
      data.lotto_list.reduce((acc, { group, country, slug, name }) => {
        if (!acc[group]) {
          acc[group] = { group: group, lotto_list: [] };
        }
        acc[group].lotto_list.push({ slug, name, country });
        return acc;
      }, {})
    );
  } catch (error) {
    console.log(error);
  }
}
</script>
