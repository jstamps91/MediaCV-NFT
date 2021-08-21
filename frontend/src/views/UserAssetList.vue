<template>
  <div class="mx-6 mt-6 py-6 px-16">
    <AssetTitle title="My NFTs" />
    <h4
      v-if="assets.length === 0"
      class="text-xl text-center pt-8"
    >
      You have no NFTs!
    </h4>
    <div class="flex flex-wrap justify-center sm:justify-start">
      <div
        v-for="asset in assets"
        :key="asset['guid']"
      >
        <AssetListCard :asset="asset" />
      </div>
    </div>
  </div>
</template>
<script>
import AssetListCard from '@/components/cards/AssetListCard';
import AssetTitle from '@/components/AssetTitle';
import { internalService } from '@/services/internal';
import { mapGetters } from 'vuex';
import eventBus from '@/utils/eventBus';
import authOnly from '@/mixins/authOnly';

export default {
  name: 'UserAssetList',
  components: {
    AssetListCard,
    AssetTitle
  },
  mixins: [authOnly],
  data() {
    return {
      assets: []
    };
  },
  computed: {
    ...mapGetters({
      userAssets: 'algorand/userAssets',
      account: 'algorand/account'
    })
  },
  watch: {
    userAssets() {
      this.fetchAssetList();
    }
  },
  async mounted() {
    await this.fetchAssetList();
    eventBus.$on('update', this.fetchAssetList);
  },
  beforeDestroy() {
    eventBus.$off('update');
  },
  methods: {
    async fetchAssetList() {
      const ownedAssets = Object.keys(this.userAssets).filter(key => {
        return this.userAssets[key].amount > 0;
      });
      const response = await internalService.getAssets({
        status: 'RD',
        ordering: 'created_at',
        asset_id__in: ownedAssets,
        owner_address: this.account
      });
      this.assets = response['results'];
    }
  }
};
</script>
