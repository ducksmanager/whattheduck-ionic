<template>
  <List
    :items="sortedItems"
    :fill-percentages="ownershipPercentages"
    :get-target-route-fn="getTargetUrlFn"
    :get-item-text-fn="getItemTextFn"
  >
    <template #row-label="{ item }">
      <Country :id="item.countrycode" :label="item.countryname" />
    </template>
    <template #row-suffix="{ item }" v-if="ownershipPercentages">
      {{ getOwnershipText(ownershipPercentages[item.countrycode]) }}
    </template>
  </List>
</template>

<script setup lang="ts">
import { stores } from '~web';

import { getOwnershipText, getOwnershipPercentages } from '~/composables/useOwnership';
import { app } from '~/stores/app';
import { wtdcollection } from '~/stores/wtdcollection';

const router = useRouter();
const route = useRoute();
const collectionStore = wtdcollection();
const coaStore = stores.coa();
const appStore = app();

const totalPerCountry = computed(() => collectionStore.totalPerCountry);
const issueCountsPerCountry = computed(() => coaStore.issueCountsPerCountry!);

const ownershipPercentages = computed(() =>
  getOwnershipPercentages(totalPerCountry.value, issueCountsPerCountry.value),
);

const items = computed(() =>
  coaStore.countryNames
    ? Object.entries(coaStore.countryNames)
        .filter(([countrycode]) => appStore.isCoaView || collectionStore.ownedCountries!.includes(countrycode))
        .map(([countrycode, countryname]) => ({
          key: countrycode,
          item: { countrycode, countryname },
        }))
    : [],
);

const getItemTextFn = (item: (typeof items)['value'][0]['item']) => item.countryname || item.countrycode;

const sortedItems = computed(() =>
  [...items.value].sort(({ item: { countryname: text1 } }, { item: { countryname: text2 } }) =>
    text1.toLowerCase().localeCompare(text2.toLowerCase()),
  ),
);

const getTargetUrlFn = (key: string) => ({
  name: 'PublicationList',
  params: { type: route.params.type, countrycode: key },
});

collectionStore.fetchAndTrackCollection().catch(() => {
  router.push('/');
});
</script>
