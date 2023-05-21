<template>
  <List v-if="hasCoaData" :items="sortedItems" :get-target-url-fn="getTargetUrlFn">
    <template #row-prefix="{ item }">
      <ion-checkbox v-if="isCoaList"></ion-checkbox>
      <Condition :value="getConditionKey(item)" />
    </template>
    <template #row-label="{ text }">
      <Issue :value="text" />
    </template>
  </List>
</template>

<script setup lang="ts">
import { IonCheckbox } from '@ionic/vue';
import Issue from '~/components/Issue.vue';
import Condition from '~/components/Condition.vue';
import List from '~/components/List.vue';
import { IssueWithPublicationcode, collection } from '~/stores/collection';
import { computed } from 'vue';
import { condition } from '~/stores/condition';
import { coa } from '~/stores/coa';
import { useRoute } from 'vue-router';
import { watch } from 'vue';
import { app } from '~/stores/app';

const route = useRoute();

const collectionStore = collection();
const coaStore = coa();
const conditionStore = condition();
const appStore = app();

const isCoaList = computed(() => route.params.type === 'coa');
const conditionL10n = computed(() => conditionStore.conditionL10n);

const hasCoaData = computed(() => !!coaStore.issueNumbers?.[publicationcode.value]);

const getTargetUrlFn = (routePath: string, key: string) =>
  `/collection/${key.replace(routePath, '').replace(' ', '/')}`;

const getConditionKey = (item: IssueWithPublicationcode) =>
  conditionL10n.value.find(({ fr }) => fr === item.condition)?.en || 'none';

const publicationcode = computed(() => `${route.params.countrycode}/${route.params.magazinecode}`);

watch(
  () => publicationcode.value,
  async (newValue) => {
    appStore.currentNavigationItem = newValue as string;
  },
  { immediate: true }
);

const items = computed(() =>
  (collectionStore.collection || [])
    .filter((issue) => issue.publicationcode === publicationcode.value)
    .map(({ issuenumber, ...issue }) => ({
      key: `${issue.publicationcode} ${issuenumber}`,
      text: issuenumber,
      ...issue,
    }))
);

const sortedItems = computed(() => {
  const keys = items.value.map(({ key }) => key);
  return coaStore.issueNumbers[publicationcode.value || '']
    .filter((issuenumber) => keys.includes(`${publicationcode.value} ${issuenumber}`))
    .map((issuenumber) => ({
      key: `${publicationcode.value} ${issuenumber}`,
      text: issuenumber,
      ...items.value.find(({ key }) => key === `${publicationcode.value} ${issuenumber}`),
    }));
});

collection().loadCollection();
coaStore.fetchIssueNumbers([publicationcode.value]);
</script>