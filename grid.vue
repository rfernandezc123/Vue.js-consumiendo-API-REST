<template>
  <div>
    <filter-slot
      class="filter-slot-new-customization"
      :filter="filterSlot.Filters"
      :total-rows="filterSlot.totalRows"
      :paginate="filterSlot.paginate"
      :start-page="filterSlot.startPage"
      :to-page="filterSlot.toPage"
      :filter-principal="filterSlot.filterPrincipal"
      :top-pagination="true"
      @reload="$refs['meta-ads-table'].refresh()"
    >
      <b-table
        id="meta-ads-table"
        ref="meta-ads-table"
        slot="table"
        class="position-relative table-new-customization"
        empty-text="No matching records found"
        sticky-header="60vh"
        primary-key="id"
        responsive="md"
        show-empty
        no-border-collapse
        no-provider-filtering
        :busy.sync="isBusy"
        :fields="Fields"
        :items="myProvider"
        :per-page="filterSlot.paginate.perPage"
        :current-page="filterSlot.paginate.currentPage"
      >
        <template #table-busy>
          <div class="text-center text-primary my-2">
            <b-spinner class="align-middle mr-1" />
            <strong>Loading ...</strong>
          </div>
        </template>
        <template #cell(ad_name)="data">
          <div
            v-b-tooltip.hover="
              data.item.ad_name.length > 20 ? data.item.ad_name : ''
            "
            class="position-relative h-100 w-100"
          >
            <b-badge
              v-if="!data.item.exists_in_soft"
              variant="light-danger"
              class="only-meta-badge"
            >
              Only Meta
            </b-badge>
            <div
              :class="{
                'cursor-pointer text-primary text-decoration-underline':
                  data.item.ad_route,
                'text-secondary': !data.item.ad_route,
                'pt-1': !data.item.exists_in_soft,
              }"
              @click="data.item.ad_route && getAdImagePreviewInSoft(data.item)"
            >
              {{ data.item.ad_name | limitChars(20) }}
            </div>
            <div>#{{ data.item.ad_id }}</div>
          </div>
        </template>
        <template #cell(campaign_id)="data">
          <div>#{{ data.item.campaign_id }}</div>
          <div>({{ data.item.campaign_name }})</div>
        </template>
        <template #cell(business_account_name)="data">
          <div>#{{ data.item.business_account_id }}</div>
          <div>({{ data.item.business_account_name }})</div>
        </template>
        <template #cell(spend)="data">
          <span>{{ data.item.spend | formatCurrency }}</span>
        </template>
        <template #cell(cost_per_result)="data">
          <span>{{ data.item.cost_per_result | formatCurrency }}</span>
        </template>
        <template #cell(ad_type)="data">
          <b-badge
            :variant="getVariantByAdType(data.item.ad_type)"
            :class="{
              'cursor-pointer': data.item.url_flyer_meta || data.item.video_id,
            }"
            @click="handleAdType(data.item)"
          >
            {{ data.item.ad_type }}
            <feather-icon
              v-if="data.item.url_flyer_meta || data.item.video_id"
              icon="ArrowUpRightIcon"
              size="20"
            />
          </b-badge>
        </template>
        <template #cell(program_name)="data">
          <div
            v-if="data.item.program_name"
            class="d-flex justify-content-center align-items-center"
          >
            <div class="programs_image">
              <img
                :src="getProgramImageRoute(data.item.program_id)"
                v-b-tooltip.right="data.item.program_name"
                alt="img_services"
              />
            </div>
          </div>
          <div v-else>-</div>
        </template>
        <template #cell(preview_ad)="data">
          <feather-icon
            icon="EyeIcon"
            size="20"
            class="cursor-pointer text-primary"
            @click="getAdPreview(data.item)"
          />
        </template>
        <template #bottom-row>
          <b-th colspan="2" />
          <b-th>
            <b-badge
              variant="primary"
              class="w-100 text-center"
              style="font-size: 14px; font-weight: 600"
            >
              TOTAL
            </b-badge>
          </b-th>
          <b-th>
            <b-badge
              variant="primary"
              class="w-100 text-center"
              style="font-size: 14px; font-weight: 600"
            >
              {{ totalReach }}
            </b-badge>
          </b-th>
          <b-th>
            <b-badge
              variant="primary"
              class="w-100 text-center"
              style="font-size: 14px; font-weight: 600"
            >
              {{ totalSpend | formatCurrency }}
            </b-badge>
          </b-th>
          <b-th>
            <b-badge
              variant="primary"
              class="w-100 text-center"
              style="font-size: 14px; font-weight: 600"
            >
              {{ totalResults }}
            </b-badge>
          </b-th>
          <b-th>
            <b-badge
              variant="primary"
              class="w-100 text-center"
              style="font-size: 14px; font-weight: 600"
            >
              {{ totalCostPerResult | formatCurrency }}
            </b-badge>
          </b-th>
          <b-th class="fill-bottom-row" colspan="4" />
        </template>
      </b-table>
    </filter-slot>
    <preview-ad
      v-if="showPreviewAd"
      :type="type"
      :ad-id="adId"
      :video-id="videoId"
      :ad-name="adName"
      :meta-user-id="metaUserId"
      @hidden="showPreviewAd = false"
    />
  </div>
</template>
  <script>
import MetaService from "@/views/creative/views/meta/service/meta.service.js";
import Filters from "@/views/creative/views/meta/data/filters.data.js";
import Fields from "@/views/creative/views/meta/data/fields.data.js";
import PreviewAd from "@/views/creative/views/meta/components/PreviewAd.vue";
export default {
  name: "MetaMain",
  components: {
    PreviewAd,
  },
  filters: {
    formatCurrency(value) {
      if (typeof value === "string") {
        value = value.replace(/,/g, "");
      }

      value = parseFloat(value);
      if (!value || value === 0 || isNaN(value)) {
        return "$0.00";
      }
      return value.toLocaleString(
        "en-US",
        {
          style: "currency",
          currency: "USD",
        },
        { minimumFractionDigits: 2 },
        { maximumFractionDigits: 2 }
      );
    },
  },
  data() {
    return {
      isBusy: false,
      Fields,
      filterSlot: {
        Filters,
        paginate: {
          currentPage: 1,
          perPage: 50,
        },
        startPage: 0,
        toPage: 0,
        totalRows: 0,
        filterPrincipal: {
          type: "input",
          inputType: "text",
          placeholder: "Search by ad name or campaign id or Ad id",
          model: "",
        },
      },
      totalReach: 0,
      totalSpend: 0,
      totalCostPerResult: 0,
      totalResults: 0,
      showPreviewAd: false,
    };
  },
  computed: {},
  created() {},
  mounted() {},
  methods: {
    async myProvider(ctx) {
      try {
        const orderBy = ctx.sortBy ? ctx.sortBy : "date";
        const order = ctx.sortDesc ? "desc" : ctx.sortBy ? "asc" : "desc";

        const params = {
          search_text: this.filterSlot.filterPrincipal.model,
          ad_type: this.getFilterValueByKey("ad_type"),
          date_from: this.getFilterValueByKey("date_from"),
          date_to: this.getFilterValueByKey("date_to"),
          program_id: this.getFilterValueByKey("program_id"),
          npage: this.filterSlot.paginate.currentPage,
          perpage: this.filterSlot.paginate.perPage,
          orderby: orderBy,
          order: order,
        };

        const { data } = await MetaService.geAdsInsights(params);
        this.filterSlot.totalRows = data.total;
        this.filterSlot.startPage = data.from ?? 0;
        this.filterSlot.toPage = data.to ?? 0;
        this.filterSlot.paginate.currentPage = data.current_page;
        this.filterSlot.paginate.perPage = data.per_page;
        this.totalReach = data.data?.[0]?.total_reach ?? 0;
        this.totalSpend = data.data?.[0]?.total_spend ?? 0;
        this.totalCostPerResult = data.data?.[0]?.total_cost_per_result ?? 0;
        this.totalResults = data.data?.[0]?.total_results ?? 0;
        return data.data;
      } catch (error) {
        this.showErrorSwal(error);
      }
    },
    getFilterValueByKey(key) {
      return this.filterSlot.Filters.find((filter) => filter.key === key).model;
    },
    getVariantByAdType(adType) {
      const types = {
        Flyer: "light-primary",
        Video: "light-success",
        Text: "light-info",
        Other: "light-secondary",
      };
      return types[adType] ?? "light-secondary";
    },
    getAdImagePreviewInSoft(item) {
      this.resetVariables();
      this.showPreviewAd = true;
      this.type = "preview_image_in_soft";
      this.adId = item.ad_id;
      this.adName = item.ad_name;
    },
    getAdPreview(item) {
      this.resetVariables();
      this.showPreviewAd = true;
      this.adId = item.ad_id;
      this.metaUserId = item.meta_user_id;
      this.adName = item.ad_name;
      this.type = "ad_preview";
    },
    handleAdType(item) {
      if (item.url_flyer_meta) {
        this.getAdImagePreviewInMeta(item);
      } else if (item.video_id) {
        this.getAdVideoPreview(item);
      }
    },
    getAdImagePreviewInMeta(item) {
      this.resetVariables();
      this.showPreviewAd = true;
      this.type = "preview_image_in_meta";
      this.adId = item.ad_id;
      this.metaUserId = item.meta_user_id;
      this.adName = item.ad_name;
    },
    getAdVideoPreview(item) {
      this.resetVariables();
      this.showPreviewAd = true;
      this.type = "preview_video_in_meta";
      this.videoId = item.video_id;
      this.adName = item.ad_name;
      this.metaUserId = item.meta_user_id;
    },
    resetVariables() {
      this.metaUserId = null;
      this.adName = null;
      this.type = null;
      this.adId = null;
      this.videoId = null;
    },
  },
};
</script>
<style lang="scss">
#meta-ads-table {
  .only-meta-badge {
    position: absolute;
    left: -27px;
    top: -15px;
  }
  .text-decoration-underline {
    text-decoration: underline !important;
  }
  .programs_image {
    background-color: #e9e9e9;
    padding: 0.2rem;
    border-radius: 10px;
    width: 2.5rem;
    height: 2.5rem;
    overflow: hidden;

    img {
      width: 100%;
      height: 100%;
    }
  }
  .b-table-bottom-row {
    position: sticky;
    bottom: 0;
    z-index: 1;

    th:nth-child(1) {
      border-radius: 0px 0px 0px 15px !important;
      margin-left: 5px !important;
      background: #0090e7 !important;
    }

    th:nth-child(n + 2):nth-child(-n + 6) {
      font-size: 16px;
      padding-top: 10px;
      padding-bottom: 10px;
      background: #0090e7 !important;
    }

    th:nth-child(7) {
      border-radius: 0px 0px 15px 0px !important;
      margin-left: 10px !important;
      background: #0090e7 !important;
    }
  }
  .fill-bottom-row {
    background-color: #ffffff !important;
  }
}

.dark-layout {
  #meta-ads-table {
    .fill-bottom-row {
      background-color: #17171a !important;
      border-top: none !important;
    }
  }
}
</style>
