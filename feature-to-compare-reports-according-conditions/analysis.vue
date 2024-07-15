<template>
    <div class="analysis-page">
        <a-page-header :ghost="false" :title="`${_.upperFirst(serviceType)} Analysis`" :breadcrumb="breadcrumb" class="service-type-analysis-header">
            <template #extra>
              <a-switch v-model:checked="showComparisonFilters">
                <template #checkedChildren>
                  <span title="Cancel Comparison">Cancel</span>
                </template>
                <template #unCheckedChildren>
                  <span title="Enable Comparison">Comparison</span>
                </template>
              </a-switch>
              <a-space v-if="!showComparisonFilters">
                <Filters
                  :initPath="initPath"
                  :loading="loading"
                  :windowWidth="800"
                  @refetchData="handleRefetchData"
                >
                  <template v-slot:button-first>
                    <a-select
                      class="select-service-type"
                      v-model:value="dataServiceType"
                      show-search
                      :allowClear="true"
                      placeholder="Please choose a service type"
                      :options="optionsService"
                      :filter-option="filterOption"
                      @select="handlSelectServiceId"
                      :loading="loading"
                      :disabled="loading"
                    />
                  </template>
                </Filters>
              </a-space>
              <a-popover placement="bottom" v-if="showComparisonFilters">
                <template #content>
                  Compare <b>{{ count }}</b> report{{ count > 1 ? 's' : '' }}
                  <a-divider style="margin: 4px 0 0 0"></a-divider>
                  <div>
                    <a-checkbox style="padding-left: 0; margin: 4px 0" v-model:checked="sameService">Same service</a-checkbox>
                  </div>
                  <a-divider style="margin: 4px 0 0 0"></a-divider>
                  <div>
                    <a-button
                      style="padding-left: 0"
                      type="text"
                      @click="changeReportQuantity('add')"
                      :disabled="(count === 1
                        && sameService
                        && optionsService.find(option => option.label === dataFirst?.service)?.value !== dataServiceType)
                      || count === 2
                      || loading"
                    >
                      <a-space class="title-span" title="Thêm Thống kê hiện tại vào Bảng so sánh">
                        <PlusSquareOutlined />
                        Add
                      </a-space>
                    </a-button>
                  </div>
                  <a-divider style="margin: 0"></a-divider>
                  <div>
                    <a-button
                      style="padding-left: 0"
                      type="text"
                      @click="changeReportQuantity('remove')"
                      :disabled="count === 0 || loading"
                    >
                      <a-space class="title-span" title="Xóa Thống kê hiện tại khỏi Bảng so sánh">
                        <MinusSquareOutlined />
                        Remove
                      </a-space>
                    </a-button>
                  </div>
                </template>
                <a-space>
                  <Filters
                    :initPath="initPath"
                    :loading="loading"
                    :windowWidth="800"
                    @refetchData="handleRefetchData"
                    :extraDisabled="!sameService && count >= 1"
                  >
                    <template v-slot:button-first>
                      <a-select
                        class="select-service-type"
                        v-model:value="dataServiceType"
                        show-search
                        :allowClear="true"
                        placeholder="Please choose a service type"
                        :options="optionsService"
                        :filter-option="filterOption"
                        @select="handlSelectServiceId"
                        :loading="loading"
                        :disabled="sameService || loading"
                      />
                    </template>
                  </Filters>
                  <a-button type="primary" @click="submitComparison" :disabled="count < 2 || loading">
                    <b>Compare</b>
                  </a-button>
                </a-space>
              </a-popover>
              <!--                    Modal-->
              <a-modal
                v-model:visible="visibleModalOrderReports"
                width="100%"
                wrap-class-name="full-modal"
                :footer="null"
              >
                <template #title>
                  <a-space class="title-span">
                    <DashboardOutlined />
                    SO SÁNH THỐNG KÊ ĐƠN HÀNG
                  </a-space>
                </template>
                <a-row :gutter="[12, 12]">
                  <a-col :span="24">
                    <a-card :bordered="false">
                      <template #title>
                        <a-space>
                          <DollarOutlined />
                          COMPARISON TOTAL NUMBERS
                        </a-space>
                      </template>
                      <a-card-grid style="width: 50%">
                        <ChartStacked
                            direction="y"
                            chartType="bar"
                            :data="dataAll?.totalOrder"
                            :axisObject="axisObjectAll?.totalOrder"
                        />
                      </a-card-grid>
                      <a-card-grid style="width: 50%">
                        <ChartStacked
                            direction="y"
                            chartType="bar"
                            :data="dataAll?.totalRevenue"
                            :axisObject="axisObjectAll?.totalRevenue"
                        />
                      </a-card-grid>
                      <a-card-grid style="width: 50%">
                        <ChartStacked
                            direction="y"
                            chartType="bar"
                            :data="dataAll?.totalRefund"
                            :axisObject="axisObjectAll?.totalRefund"
                        />
                      </a-card-grid>
                      <a-card-grid style="width: 50%">
                        <ChartStacked
                            direction="y"
                            chartType="bar"
                            :data="dataAll?.totalProcessingFee"
                            :axisObject="axisObjectAll?.totalProcessingFee"
                        />
                      </a-card-grid>
                      <a-card-grid style="width: 50%">
                        <ChartStacked
                            direction="y"
                            chartType="bar"
                            :data="dataAll?.totalGovFee"
                            :axisObject="axisObjectAll?.totalGovFee"
                        />
                      </a-card-grid>
                      <a-card-grid style="width: 50%">
                        <ChartStacked
                            direction="y"
                            chartType="bar"
                            :data="dataAll?.totalServiceFee"
                            :axisObject="axisObjectAll?.totalServiceFee"
                        />
                      </a-card-grid>
                    </a-card>
                  </a-col>
                  <a-col :span="24">
                    <a-card :bordered="false">
                      <template #title>
                        <a-space>
                          <LineChartOutlined />
                          COMPARISON ORDERS
                        </a-space>
                      </template>
                      <ChartStacked
                          chartType="bar"
                          :data="dataAll?.orderStatisticsChart?.totalOrder"
                          :axisObject="axisObjectAll?.orderStatisticsChart?.totalOrder"
                      />
                    </a-card>
                  </a-col>
                  <a-col :span="24">
                    <a-card :bordered="false">
                      <template #title>
                        <a-space>
                          <LineChartOutlined />
                          COMPARISON RANGE NUMBERS
                        </a-space>
                      </template>
                      <ChartStacked
                          chartType="bar"
                          :data="dataAll?.orderStatisticsChart?.totalRevenue"
                          :axisObject="axisObjectAll?.orderStatisticsChart?.totalRevenue"
                      />
                    </a-card>
                  </a-col>
                </a-row>
              </a-modal>
            </template>
        </a-page-header>
        <div class="page-wrapper">
            <div class="service-type-analysis-page" v-if="serviceType === 'evisa'">
                <a-row class="first-row" :gutter="12">
                    <a-col :xs="12" :sm="12" :md="4" :lg="4" :xl="4">
                        <a-card :bordered="false" class="analysis-tabs">
                            <a-statistic
                                title="Total order"
                                :value="result?.orderStatisticsBlock?.totalOrder"
                                :precision="0"
                                prefix=""
                                :value-style="{ color: 'black' }"
                            >
                            </a-statistic>
                        </a-card>
                    </a-col>
                    <a-col :xs="12" :sm="12" :md="4" :lg="4" :xl="4">
                        <a-card :bordered="false" class="analysis-tabs">
                            <a-statistic
                                title="Total Revenue"
                                :value="result?.orderStatisticsBlock?.totalRevenue"
                                :precision="0"
                                prefix="$"
                                :value-style="{ color: 'black' }"
                            >
                            </a-statistic>
                        </a-card>
                    </a-col>
                    <a-col :xs="12" :sm="12" :md="4" :lg="4" :xl="4">
                        <a-card :bordered="false" class="analysis-tabs">
                            <a-statistic
                                title="Total Refund"
                                :value="result?.orderStatisticsBlock?.totalRefund"
                                :precision="0"
                                prefix="$"
                                :value-style="{ color: 'black' }"
                            >
                            </a-statistic>
                        </a-card>
                    </a-col>
                    <a-col :xs="12" :sm="12" :md="4" :lg="4" :xl="4">
                        <a-card :bordered="false" class="analysis-tabs">
                            <a-statistic
                                title="Total Processing Fee"
                                :value="result?.orderStatisticsBlock?.totalProcessingFee"
                                :precision="0"
                                prefix="$"
                                :value-style="{ color: 'black' }"
                            >
                            </a-statistic>
                        </a-card>
                    </a-col>
                    <a-col :xs="12" :sm="12" :md="4" :lg="4" :xl="4">
                        <a-card :bordered="false" class="analysis-tabs">
                            <a-statistic
                                title="Total Gov Fee"
                                :value="result?.orderStatisticsBlock?.totalGovFee"
                                :precision="0"
                                prefix="$"
                                :value-style="{ color: 'black' }"
                            >
                            </a-statistic>
                        </a-card>
                    </a-col>
                    <a-col :xs="12" :sm="12" :md="4" :lg="4" :xl="4">
                        <a-card :bordered="false" class="analysis-tabs">
                            <a-statistic
                                    title="Total Service Fee"
                                    :value="result?.orderStatisticsBlock?.totalServiceFee"
                                    :precision="0"
                                    prefix="$"
                                    :value-style="{ color: 'black' }"
                            >
                            </a-statistic>
                        </a-card>
                    </a-col>
                </a-row>
                <a-row class="second-row" :gutter="12">
                    <a-col :span="24">
                        <a-card :bordered="false" class="analysis-tabs">
                            <a-tabs v-model:activeKey="activeKey" type="card">
                                <a-tab-pane key="1">
                                  <template #tab>
                                    <BarChartOutlined />
                                    Orders chart
                                  </template>
                                  <ChartBar :data="dataChartBarOrder" :axisObject="axisObject" />
                                </a-tab-pane>
                                <a-tab-pane key="2">
                                  <template #tab>
                                    <BarChartOutlined />
                                    Revenue chart
                                  </template>
                                  <ChartBar :data="dataChartBarRevenue" :axisObject="axisObject" />
                                </a-tab-pane>
                            </a-tabs>
                        </a-card>
                    </a-col>
                </a-row>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import {
  PlusSquareOutlined,
  MinusSquareOutlined,
  DashboardOutlined,
  LineChartOutlined,
  BarChartOutlined,
  DollarOutlined
} from "@ant-design/icons-vue";
import _ from 'lodash';
import dayjs from "dayjs";
import { routerPushObject } from "~/helpers/helpers";
import {useGlobal} from "~/stores/global";

const { services } = useGlobal();
const router = useRouter();
const route = useRoute();
const query = route.query;
const serviceType: string = route.params.serviceType;
const serviceId: Ref<String> = ref('');
const pathPrefix = `/crm/servicetype-${serviceType}`;
const initPath = `${pathPrefix}/analysis`;

const breadcrumb = {
    routes: [
        {
            path: `${pathPrefix}`,
            breadcrumbName: `Service ${_.upperFirst(serviceType)}`,
        },
        {
            path: `${pathPrefix}/analysis`,
            breadcrumbName: 'Analysis',
        },
    ],
};

const optionsService = services
    ?.filter(service => service.type === serviceType.toUpperCase())
    ?.map(service => ({
        label: service.domain,
        value: service.id,
    }));
const dataServiceType = ref();
const filterOption = (input, option) => {
    return option.label.toLowerCase().indexOf(input.toLowerCase()) >= 0;
};
const handlSelectServiceId = async (id: string) => {
    serviceId.value = id;
    refetch({
        serviceId: serviceId.value,
    })
};

const activeKey = ref('1');

const calendarValue = ref(query.time_start && query.time_end
    ? [dayjs(query.time_start), dayjs(query.time_end)]
    : [dayjs(), dayjs()]
);

const variblesForStatisticsServiceType = computed(() => ({
    dateRange: {
        from: dayjs(query.time_start ? query.time_start : calendarValue.value[0]).format('YYYY-MM-DD'),
        to: dayjs(query.time_end ? query.time_end : calendarValue.value[1]).format('YYYY-MM-DD'),
    },
    serviceType: serviceType.toUpperCase(),
}));

const queryStatistics= gql`
    query orderStatistics($dateRange: DateRange, $serviceId: ID, $serviceType: ServiceType) {
        orderStatisticsBlock(dateRange: $dateRange, serviceId: $serviceId, serviceType: $serviceType) {
            totalOrder
            totalRevenue
            totalRefund
            totalProcessingFee
            totalGovFee
            totalServiceFee
        }
        orderStatisticsChart(dateRange: $dateRange, serviceId: $serviceId, serviceType: $serviceType) {
            totalOrder {
                date
                value
            }
            totalRevenue {
                date
                value
            }
        }
    }
`
const { result, loading, refetch } = useQuery(queryStatistics, variblesForStatisticsServiceType.value, {
    fetchPolicy: 'no-cache',
});

const customValue = ref(calendarValue.value.map(val => val.format('YYYY-MM-DD')))
const handleRefetchData = (data: Object) => {
    const titleTime = data?.selectTimeQuickly;
    const startTime = dayjs(data?.timeRange[0]).format('YYYY-MM-DD');
    const endTime = dayjs(data?.timeRange[1]).format('YYYY-MM-DD');
    customValue.value = [startTime, endTime];
    router.push(routerPushObject(initPath, null, null, titleTime, startTime, endTime));
    refetch(data?.variables);
};

const axisObject = ref({
  xText: '',
  yText: '',
});
const dataChartBarRevenue = ref({
  labels: [],
  datasets: [],
});
const dataChartBarOrder = ref({
  labels: [],
  datasets: [],
});
watch(result, () => {
  if(result) {
    const dataRevenue = result.value?.orderStatisticsChart?.totalRevenue;
    const dataOrder = result.value?.orderStatisticsChart?.totalOrder;
    dataChartBarRevenue.value = {
      labels: dataRevenue?.map(stat => stat.date),
      datasets: [{
        label: 'Revenue ($)',
        backgroundColor: '#1890ff',
        data: dataRevenue?.map(stat => stat.value),
      }],
    };
    dataChartBarOrder.value = {
      labels: dataOrder?.map(stat => stat.date),
      datasets: [{
        label: 'Order',
        backgroundColor: '#1890ff',
        data: dataOrder?.map(stat => stat.value),
      }],
    };
  }
});

// COMPARE REPORTS
const visibleModalOrderReports = ref(false);
const showComparisonFilters = ref(false);
const sameService = ref(false);
const count = ref(0);
let dataAll;
let dataFirst;
let dataSecond;
const changeReportQuantity = (action: string) => {
  if(action === 'add') {
    count.value++;
    localStorage.setItem(`report${count.value}`, JSON.stringify(Object.assign(
        {},
        result.value,
        {
          time: customValue.value,
          duration: (dayjs(customValue.value[1]) - dayjs(customValue.value[0]))/86400000,
        },
        {
          service: optionsService.find(option => option.value === dataServiceType.value)?.label
        }
      )));
  } else if(action === 'remove') {
    count.value--;
    localStorage.removeItem(`report${count.value+1}`);
  }
};
const submitComparison = () => {
  visibleModalOrderReports.value = true;
  dataFirst = JSON.parse(localStorage.getItem(`report1`));
  dataSecond = JSON.parse(localStorage.getItem(`report2`));

  let dataLabelForBlocks = !sameService
    ? [dataFirst?.service, dataSecond?.service]
    : dataFirst?.duration === 0
      ? dataFirst?.orderStatisticsChart?.time[0]
      : dataFirst?.orderStatisticsChart?.time?.join(' - ');
  let dataLabelForChart = dataFirst?.orderStatisticsChart?.totalOrder?.map(val => val?.date);

  dataAll = Object.assign(
    {},
    {
      totalOrder: {
        labels: dataLabelForBlocks,
        datasets: [
          {
            label: dataFirst?.service,
            borderColor: 'DarkOrange',
            backgroundColor: 'Orange',
            data: dataFirst?.orderStatisticsBlock?.totalOrder,
          },
          {
            label: dataSecond?.service,
            borderColor: 'Blue',
            backgroundColor: 'DodgerBlue',
            data: dataSecond?.orderStatisticsBlock?.totalOrder,
          },
        ],
      },
      totalRevenue: {
        labels: dataLabelForBlocks,
        datasets: [
          {
            label: dataFirst?.service,
            borderColor: 'DarkOrange',
            backgroundColor: 'Orange',
            data: dataFirst?.orderStatisticsBlock?.totalRevenue,
          },
          {
            label: dataSecond?.service,
            borderColor: 'Blue',
            backgroundColor: 'DodgerBlue',
            data: dataSecond?.orderStatisticsBlock?.totalRevenue,
          },
        ],
      },
      totalRefund: {
        labels: dataLabelForBlocks,
        datasets: [
          {
            label: dataFirst?.service,
            borderColor: 'DarkOrange',
            backgroundColor: 'Orange',
            data: dataFirst?.orderStatisticsBlock?.totalRefund,
          },
          {
            label: dataSecond?.service,
            borderColor: 'Blue',
            backgroundColor: 'DodgerBlue',
            data: dataSecond?.orderStatisticsBlock?.totalRefund,
          },
        ],
      },
      totalProcessingFee: {
        labels: dataLabelForBlocks,
        datasets: [
          {
            label: dataFirst?.service,
            borderColor: 'DarkOrange',
            backgroundColor: 'Orange',
            data: dataFirst?.orderStatisticsBlock?.totalProcessingFee,
          },
          {
            label: dataSecond?.service,
            borderColor: 'Blue',
            backgroundColor: 'DodgerBlue',
            data: dataSecond?.orderStatisticsBlock?.totalProcessingFee,
          },
        ],
      },
      totalGovFee: {
        labels: dataLabelForBlocks,
        datasets: [
          {
            label: dataFirst?.service,
            borderColor: 'DarkOrange',
            backgroundColor: 'Orange',
            data: dataFirst?.orderStatisticsBlock?.totalGovFee,
          },
          {
            label: dataSecond?.service,
            borderColor: 'Blue',
            backgroundColor: 'DodgerBlue',
            data: dataSecond?.orderStatisticsBlock?.totalGovFee,
          },
        ],
      },
      totalServiceFee: {
        labels: dataLabelForBlocks,
        datasets: [
          {
            label: dataFirst?.service,
            borderColor: 'DarkOrange',
            backgroundColor: 'Orange',
            data: dataFirst?.orderStatisticsBlock?.totalServiceFee,
          },
          {
            label: dataSecond?.service,
            borderColor: 'Blue',
            backgroundColor: 'DodgerBlue',
            data: dataSecond?.orderStatisticsBlock?.totalServiceFee,
          },
        ],
      },
      orderStatisticsChart: {
        totalOrder: {
          labels: dataLabelForChart,
          datasets: [
            {
              label: dataFirst?.service,
              borderColor: 'DarkOrange',
              backgroundColor: 'Orange',
              data: dataFirst?.orderStatisticsChart?.totalOrder?.map(val => val?.value),
            },
            {
              label: dataSecond?.service,
              borderColor: 'Blue',
              backgroundColor: 'DodgerBlue',
              data: dataSecond?.orderStatisticsChart?.totalOrder?.map(val => val?.value),
            },
          ],
        },
        totalRevenue: {
          labels: dataLabelForChart,
          datasets: [
            {
              label: dataFirst?.service,
              borderColor: 'DarkOrange',
              backgroundColor: 'Orange',
              data: dataFirst?.orderStatisticsChart?.totalRevenue?.map(val => val?.value),
            },
            {
              label: dataSecond?.service,
              borderColor: 'Blue',
              backgroundColor: 'DodgerBlue',
              data: dataSecond?.orderStatisticsChart?.totalRevenue?.map(val => val?.value),
            },
          ],
        },
      },
    },
  );
};
const axisObjectAll = ref({
  totalOrder: {
    yText: '',
    xText: 'Order',
  },
  totalRevenue: {
    yText: '',
    xText: 'Revenue',
  },
  totalRefund: {
    yText: '',
    xText: 'Refund',
  },
  totalProcessingFee: {
    yText: '',
    xText: 'Processing Fee',
  },
  totalGovFee: {
    yText: '',
    xText: 'Gov Fee',
  },
  totalServiceFee: {
    yText: '',
    xText: 'Service Fee',
  },
  orderStatisticsChart: {
    totalOrder: {
      xText: '',
      yText: 'Order',
    },
    totalRevenue: {
      xText: '',
      yText: 'Revenue',
    },
  },
});
</script>

<style lang="scss">
.order-analysis-comparison-dropdown {
  .ant-dropdown-menu-item {
    padding: 0;
    .ant-btn-text {
      width: 100%;
      text-align: left;
    }
  }
}
.analysis-page{
  .service-type-analysis-header {
    .ant-page-header-heading {
      .ant-page-header-heading-left {
        .ant-page-header-heading-title {
          min-width: 240px;
          white-space: normal;
        }
      }
      .ant-page-header-heading-extra {
        display: flex;
        align-items: center;
      }
      @media (max-width: 992px) {
        display: contents;
      }
    }
    .button-second {
      .ant-btn-text {
        display: flex;
      }
    }
  }
  .service-type-analysis-page {
    .first-row{
      .analysis-tabs{
        margin-bottom: 12px;
        height: calc(100% - 12px);
      }
    }
    .ant-row:not(.ant-row:last-child) {
      margin-bottom: 12px;
    }
  }
}
</style>
