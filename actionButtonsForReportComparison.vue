<template>
    <div class="analysis-page">
        <a-page-header :ghost="false" :title="`${_.upperFirst(serviceType)} Analysis`" :breadcrumb="breadcrumb" class="service-type-analysis-header">
            <template #extra>
              <ExtrasFilters
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
                      />
                  </template>

                  <template v-slot:button-second>
                    <a-dropdown-button
                      overlayClassName="order-analysis-comparison-dropdown"
                      type="primary"
                      :loading="loading"
                      placement="right"
                    >
                      <template #overlay>
                        <a-menu>
                          <a-menu-item key="1">
                              <a-button
                                  type="text"
                                  @click="changeReportQuantity('add')"
                                  :disabled="count === 2"
                              >
                                <a-space class="title-span" title="Thêm Thống kê hiện tại vào Bảng so sánh">
                                  <PlusOutlined />
                                  Add
                                </a-space>
                              </a-button>
                          </a-menu-item>
                          <a-menu-item key="2">
                              <a-button
                                  type="text"
                                  @click="changeReportQuantity('remove')"
                                  :disabled="count === 0"
                              >
                                <a-space class="title-span" title="Xóa Thống kê hiện tại khỏi Bảng so sánh">
                                  <MinusOutlined />
                                  Remove
                                </a-space>
                              </a-button>
                          </a-menu-item>
                          <a-menu-item key="3">
                              <a-button
                                  type="text"
                                  @click="changeReportQuantity('reset')"
                                  :disabled="count === 0"
                              >
                                <a-space class="title-span" title="Xóa Tất cả dữ liệu">
                                  <UndoOutlined />
                                  Reset
                                </a-space>
                              </a-button>
                          </a-menu-item>
                        </a-menu>
                      </template>
                      <a-badge :count="count" :title="`Hiện có ${count} dữ liệu trong Bảng So sánh`">
                        <a-button
                            type="text"
                            title="So Sánh"
                            @click="compareOrderReports"
                            :disabled="count < 2"
                        >
                          <BarChartOutlined style="color: white"/>
                        </a-button>
                      </a-badge>
                    </a-dropdown-button>
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
                                  So sánh Tổng số
                                </a-space>
                              </template>
                              <a-card-grid style="width: 33.33%">
                                <ChartStacked chartType="bar" :data="dataStackedChartBar" :axisObject="axisObjectReport" />
                              </a-card-grid>
                              <a-card-grid style="width: 33.33%">
                                <ChartStacked chartType="bar" :data="dataStackedChartBar" :axisObject="axisObjectReport" />
                              </a-card-grid>
                              <a-card-grid style="width: 33.33%">
                                <ChartStacked chartType="bar" :data="dataStackedChartBar" :axisObject="axisObjectReport" />
                              </a-card-grid>
                              <a-card-grid style="width: 33.33%">
                                <ChartStacked chartType="bar" :data="dataStackedChartBar" :axisObject="axisObjectReport" />
                              </a-card-grid>
                              <a-card-grid style="width: 33.33%">
                                <ChartStacked chartType="bar" :data="dataStackedChartBar" :axisObject="axisObjectReport" />
                              </a-card-grid>
                              <a-card-grid style="width: 33.33%">
                                <ChartStacked chartType="bar" :data="dataStackedChartBar" :axisObject="axisObjectReport" />
                              </a-card-grid>
                            </a-card>
                          </a-col>
                          <a-col :span="24">
                            <a-card :bordered="false">
                              <template #title>
                                <a-space>
                                  <LineChartOutlined />
                                  So sánh Giai đoạn
                                </a-space>
                              </template>
                              <ChartStacked chartType="bar" :data="dataStackedChartBar" :axisObject="axisObjectReport" />
                            </a-card>
                          </a-col>
                        </a-row>
                    </a-modal>
                  </template>
              </ExtrasFilters>
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
    DashboardOutlined,
    LineChartOutlined,
    BarChartOutlined,
    MinusOutlined,
    PlusOutlined,
    UndoOutlined,
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

const handleRefetchData = (data: Object) => {
    const titleTime = data?.selectTimeQuickly;
    const startTime = dayjs(data?.timeRange[0]).format('YYYY-MM-DD');
    const endTime = dayjs(data?.timeRange[1]).format('YYYY-MM-DD');

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
const selectedRowKeys = ref([]);
const count = ref(0);
let data1;
let data2;
const changeReportQuantity = (action: string) => {
  if(action === 'add') {
    console.log('calendarValue', calendarValue.value);
    count.value++;
    localStorage.setItem(`report${count.value}`, JSON.stringify(result.value));
    console.log('result add', result.value);
  } else if(action === 'remove') {
    count.value--;
    localStorage.removeItem(`report${count.value}`);
    console.log('result remove', result.value);
  } else if(action === 'reset') {
    count.value = 0;
    for(let i in count.value) {
      localStorage.removeItem(`report${i}`);
    }
    console.log('result reset', result.value);
  }
};
const compareOrderReports = () => {
    visibleModalOrderReports.value = true;
    data1 = JSON.parse(localStorage.getItem(`report1`));
    data2 = JSON.parse(localStorage.getItem(`report2`));
    console.log('data1 compare', data1);
    console.log('data2 compare', data2);
};
// DATA
const dataBar = [
    {
        timePeriod: '1',
        value: 5,
    }
];
const dataStackedChartBar = {
    labels: dataBar.map(stat => stat.timePeriod),
    datasets: [
        {
            label: 'Kenya',
            borderColor: 'DarkOrange',
            backgroundColor: 'Orange',
            data: data1?.map(stat => stat.value) ?? dataBar.map(stat => stat.value),
        },
        {
            label: 'Australia',
            borderColor: 'Blue',
            backgroundColor: 'DodgerBlue',
            data: data2?.map(stat => stat.value) ?? dataBar.map(stat => stat.value),
        },
    ],
};
const axisObjectReport = ref({
    xText: '',
    yText: '',
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
