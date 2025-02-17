<template>
  <div v-if="asset" class="container p-5 is-dark">
    <HeroImage src="https://via.placeholder.com/1000" />

    <div class="columns">
      <div class="column is-two-thirds">
        <StackNavigationBar @onBack="goBack" :size="3">
          <div>
            <h1 class="mt-6 title">
              <b>{{ `${asset.address} - DAO` }}</b>
              <!-- is a b tag because it's already an h1 -->
            </h1>

            <div class="subtitle mt-2 mb-2 is-6">
              ({{ numberFormat.format(asset.owners.size) }} holders,
              {{ numberFormat.format(openProposalCount) }} open proposals)
            </div>
          </div>
        </StackNavigationBar>

        <div class="my-1 p-1">
          <div class="help"><strong>Token</strong></div>
          <Address class="mb-3" :value="asset.contractAddress" />
        </div>

        <div class="my-1 p-1">
          <div class="help"><strong>Documents</strong></div>
          <ul>
            <li><a href="#">Contract Etherscan page</a></li>
            <li><a href="#">IPFS Document</a></li>
          </ul>
        </div>
        <div class="my-1 p-1">
          <div class="help"><strong>Tags</strong></div>
          <div class="mb-3">
            <span class="tag m-1 is-success">badge2</span>
            <span class="tag m-1 is-warning">badge3</span>
            <span class="tag m-1 is-info">badge1</span>
            <span class="tag m-1 is-danger">badge4</span>
            <span class="tag m-1 is-success">badge2</span>
            <span class="tag m-1 is-warning">badge3</span>
          </div>
        </div>

        <div class="my-1 p-1">
          <div class="help"><strong>Description</strong></div>
          <div>{{ asset.description }}</div>
        </div>

        <div
          class="button mt-2 mb-2 p-4 has-text-centered"
          style="width: 100%"
          @click="createProposal"
        >
          <a href="javascript:void(0)">
            <strong>Create a new proposal</strong></a
          >
        </div>

        <ul class="mt-5 mb-5 mr-1 ml-1" v-if="proposals">
          <li v-for="proposal in proposals" :key="proposal.id">
            <ProposalListItem :assetId="assetId" :proposal="proposal" />
          </li>
        </ul>
        <div v-else>
          <Loader :shadowless="true" />
        </div>
      </div>
      <div class="column is-one-third my-5 mx-1">
        <div class="panel m-2 p-4 is-flex is-flex-direction-column">
          <h2 class="is-size-5 has-text-weight-bold mb-4">Price graph</h2>
          <img class="image" src="https://via.placeholder.com/500x300" />
          <div class="table-container mt-2 mb-0">
            <table class="table" style="width: 100%">
              <tbody>
                <tr>
                  <th>Current rent</th>
                  <td class="has-text-right">
                    ${{ numberFormat.format(asset.currentRent) }}
                  </td>
                </tr>
                <tr>
                  <th>Market value</th>
                  <td class="has-text-right">
                    ${{ numberFormat.format(asset.marketValue) }}
                  </td>
                </tr>
                <tr>
                  <th>Living space</th>
                  <td class="has-text-right">
                    {{ numberFormat.format(asset.area) }} sqft
                  </td>
                </tr>
                <tr>
                  <th>Rooms</th>
                  <td class="has-text-right">
                    {{ asset.bedroomCount }} bed, {{ asset.bathroomCount }} bath
                  </td>
                </tr>
                <tr>
                  <th>Year built</th>
                  <td class="has-text-right">{{ asset.yearBuilt }}</td>
                </tr>
                <tr>
                  <th>Capitalization rate</th>
                  <td class="has-text-right">{{ asset.grossYieldPct }}</td>
                </tr>
                <tr>
                  <th>Total supply</th>
                  <td class="has-text-right">
                    {{ numberFormat.format(asset.numOfShares) }}
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
          <Accordion extraClasses="is-shadowless" margin="0" summary="Swap">
            <div class="field">
              <div class="subtitle is-6">
                <div>
                  <strong>1</strong> {{ asset.symbol }} =
                  <strong>{{ askPriceString }}</strong> ETH
                </div>
              </div>
            </div>
            <div class="field">
              <label for="source-token-amount" class="label">From</label>
              <div class="help has-text-grey has-text-weight-bold">
                Balance: {{ ethBalance }} ETH
              </div>
              <div class="control">
                <input
                  class="input"
                  type="number"
                  name="source-token-amount"
                  id="source-token-amount"
                  placeholder="0.0"
                  :value="orderFromString"
                  @keypress="isNumber($event)"
                  @input="orderInputUpdated(0, $event)"
                  required
                />
              </div>
            </div>
            <div class="field">
              <label for="dest-token-amount" class="label">To</label>
              <div class="help has-text-grey has-text-weight-bold">
                Balance: {{ shareBalance }}
              </div>
              <div class="control">
                <input
                  class="input"
                  type="number"
                  name="dest-token-amount"
                  id="dest-token-amount"
                  placeholder="0"
                  :value="orderToString"
                  @keypress="isNumber($event)"
                  @input="orderInputUpdated(1, $event)"
                  required
                />
              </div>
            </div>

            <Button label="Swap" extraClasses="w-full" @click="performSwap" />
          </Accordion>
        </div>

        <div class="panel m-2 p-4 is-flex is-flex-direction-column">
          <h2 class="is-size-5 has-text-weight-bold mb-4">Tokenholders</h2>
          <ul>
            <li
              v-for="tokenholder in asset.owners.entries()"
              :key="tokenholder[0]"
            >
              <div class="pt-1 pb-1 content is-vcentered">
                <Address :value="tokenholder[0]"></Address>
                <span class="m-1">
                  <strong>{{ tokenholder[1] }}</strong>
                  shares
                </span>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <div class="button mt-2 mb-2 p-4 has-text-centered" style="width: 100%">
      <a href="#"> <strong>Join the discussion</strong></a>
    </div>
  </div>
</template>

<style scoped>
.content.is-vcentered {
  display: flex;
  flex-wrap: wrap;
  align-content: center; /* used this for multiple child */
}
</style>

<script>
import { toFixedNumber } from "../../utils/common";
import { mapGetters, mapActions } from "vuex";
import StackNavigationBar from "../layout/navigation/StackNavigationBar.vue";
import Button from "../views/common/Button.vue";
import ProposalListItem from "../views/voting/ProposalListItem.vue";
import Accordion from "../utils/Accordion.vue";
import Loader from "../utils/Loader.vue";
import Address from "../views/address/Address.vue";
import HeroImage from "../utils/HeroImage.vue";

export default {
  name: "Voting",
  props: {
    assetId: {
      type: String,
      required: true,
    },
  },
  components: {
    StackNavigationBar,
    ProposalListItem,
    Accordion,
    Loader,
    Address,
    Button,
    HeroImage,
  },
  computed: {
    ...mapGetters({
      assetMap: "assetsById",
      assetProposalMap: "assetProposals",
      ethBalance: "userEthBalance",
      walletAddress: "userWalletAddress",
      assetPrices: "bestAssetPrices",
    }),

    shareBalance() {
      return this.asset.owners.get(this.walletAddress) ?? 0;
    },

    asset() {
      return this.assetMap.get(this.assetId);
    },

    proposals() {
      return this.assetProposalMap.get(this.assetId);
    },

    timestamp() {
      return Math.floor(Date.now() / 1000);
    },

    openProposalCount() {
      return this.asset.proposals.filter((p) => {
        return p.endTimestamp > this.timestamp;
      }).length;
    },

    orderToString() {
      return toFixedNumber(this.orderToValue);
    },

    orderFromString() {
      return toFixedNumber(this.orderFromValue);
    },

    askPrice() {
      var askETH = this.assetPrices.get(this.asset.id).ask;
      if (askETH) {
        askETH = askETH.toString() / Math.pow(10, 18);
      } else {
        askETH = 0.0;
      }
      return askETH;
    },
    askPriceString() {
      return toFixedNumber(this.askPrice);
    },
  },
  methods: {
    ...mapActions({
      refresh: "refreshProposalsDataForAsset",
      syncWallet: "syncWallet",
      swap: "swapToAsset",
    }),

    goBack() {
      this.$router.back();
    },

    createProposal() {
      this.$router.push(`/dao/${this.assetId}/proposals/create`);
    },
    isNumber(evt) {
      evt = evt ? evt : window.event;
      var charCode = evt.which ? evt.which : evt.keyCode;
      if (
        charCode > 31 &&
        (charCode < 48 || charCode > 57) &&
        charCode !== 46
      ) {
        evt.preventDefault();
      } else {
        return true;
      }
    },
    /* eslint-disable indent */
    orderInputUpdated(index, event) {
      switch (index) {
        case 0:
          this.orderFromValue = event.target.value;
          console.log("eth field value", event.target.value);
          this.orderToValue = this.convertToShares(
            this.orderFromValue
          ).toString();
          break;
        case 1:
          this.orderToValue = event.target.value;
          console.log("shares field value", event.target.value);
          this.orderFromValue = this.convertToETH(this.orderToValue).toString();
          break;
        default:
          break;
      }
    },
    convertToETH(shares) {
      console.log(this.askPrice);
      return shares * this.askPrice;
    },
    convertToShares(eth) {
      return eth / this.askPrice;
    },
    async performSwap() {
      await this.swap({
        asset: this.asset,
        amount: this.orderToValue,
        $toast: this.$toast,
      });
      this.orderFromValue = 0;
      this.orderToValue = 0;
    },
  },
  data() {
    return {
      numberFormat: new Intl.NumberFormat("en-US", {
        maximumSignificantDigits: 3,
      }),
      orderFromValue: "",
      orderToValue: "",
    };
  },
  mounted() {
    this.refresh({ assetId: this.assetId });
    this.syncWallet({ $toast: this.$toast });
  },
};
</script>