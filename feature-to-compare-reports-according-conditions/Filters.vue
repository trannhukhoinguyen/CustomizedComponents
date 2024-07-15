<template>
    <div class="extra-filter-wrapper">
        <a-space :direction="spaceDirection">
            <span class="button-second" >
                <slot name="button-second" />
            </span>
            <span class="button-first">
                <slot name="button-first" />
            </span>
            <span class="select-user-cover" v-if="hasUserFilter">
            <a-select
                class="select-user"
                v-model:value="dataUserSelect"
                show-search
                :allowClear="true"
                placeholder="Vui lòng chọn nhân viên"
                :loading="loadingRelatives"
                :disabled="!hasPermissionToReadUser || loadingRelatives || loading || extraDisabled"
                :options="optionUser"
                :filter-option="filterOptionUser"
                @change="handleChangeUser"
            />
        </span>
            <span class="select-time-title-cover">
                <a-select
                    class="select-time-title"
                    v-model:value="dataTimeRange"
                    show-search
                    placeholder="Lọc theo thời gian"
                    :loading="loading"
                    :disabled="loadingRelatives || loading || extraDisabled"
                    @select="handleSelectTimeRange"
                    style="width: 120px"
                >
                    <a-select-option value="custom">
                        Custom
                        <a-divider style="height: 2px; margin: 8px 0 0 0"></a-divider>
                    </a-select-option>
<!--                    <a-divider></a-divider>-->
                    <a-select-option value="today">Today</a-select-option>
                    <a-select-option value="yesterday">Yesterday</a-select-option>
                    <a-select-option value="this_week">This week</a-select-option>
                    <a-select-option value="last_7_days">Last 7 days</a-select-option>
                    <a-select-option value="last_week">Last week</a-select-option>
                    <a-select-option value="last_14_days">Last 14 days</a-select-option>
                    <a-select-option value="this_month">This month</a-select-option>
                    <a-select-option value="last_30_days">Last 30 days</a-select-option>
                    <a-select-option value="last_month">Last month</a-select-option>
                    <a-select-option value="this_year">This year</a-select-option>
                    <a-select-option value="all_time" v-if="hasAllTime">All time</a-select-option>
                </a-select>
            </span>
            <span class="range-picker-cover">
                <a-range-picker
                    class="range-picker"
                    :value="calendarValue || customValue"
                    :format="$t('dayjs.format.short_date')"
                    :disabled-date="disabledDate"
                    :loading="loading"
                    :disabled="dataTimeRange !== 'custom' || loading || loadingRelatives || extraDisabled"
                    @change="handleRangePickerChange"
                    @openChange="handleRangePickerOpenChange"
                    @calendarChange="handleRangePickerCalendarChange"
                />
            </span>
        </a-space>
    </div>
</template>

<script setup lang="ts">
import dayjs from 'dayjs';
import {useGlobal} from "~/stores/global";
import {ref} from "vue";

const emit = defineEmits(['refetchData']);
const props = defineProps({
    hasUserFilter: {
        type: Boolean,
        default: false,
    },
    hasAllTime: {
        type: Boolean,
        default: false,
    },
    extraDisabled: {
        type: Boolean,
        default: false,
    },
    loading: {
        type: Boolean,
        default: false,
    },
    hasDisbleDate: {
        type: Boolean,
        default: false,
    },
    initPath: {
        type: String,
        required: true,
    },
    windowWidth: {
        type: Number,
        default: 650,
    },
    initDateValue: {
        type: Array,
        default: ['today', [dayjs(), dayjs()]],
    },
});

const route = useRoute();
const query = route.query;
const global = useGlobal();
const me = global.me;

// Query Users
const queryRelatives = gql`
  query queryRelatives {
    userPublics(first: 10000) {
      data {
        id
        name
        nickname
      }
    }
    usersInMyDepartment {
      data{
        id
        name
        nickname
      }
    }
  }
`;
const { result: resultRelatives, loading: loadingRelatives } = useQuery(
    queryRelatives,
    {
      fetchPolicy: 'no-cache',
    },
);

let hasPermissionToReadUser = false;
if(
    me?.permissions?.map(permission => permission?.name)?.includes('read_user')
    || me?.permissions?.map(permission => permission?.name)?.includes('read_attendance_of_team')
    || me?.permissions?.map(permission => permission?.name)?.includes('read_attendance_of_department')
    || me?.permissions?.map(permission => permission?.name)?.includes('read_attendance_of_company')
){
  hasPermissionToReadUser = true;
}

const optionUser = computed(() => {
    return me?.permissions?.map(permission => permission?.name)?.includes('read_user')
        || me?.permissions?.map(permission => permission?.name)?.includes('read_attendance_of_company')
            ? resultRelatives.value?.userPublics.data?.map(user =>
                ({
                    label: user.nickname ? `${user.nickname} (${user.name})` : `(${user.name})`,
                    value: user.id,
                })
              )
            : me?.permissions?.map(permission => permission?.name)?.includes('read_attendance_of_department')
                ? resultRelatives.value?.usersInMyDepartment.data?.map(user =>
                    ({
                        label: user.nickname ? `${user.nickname} (${user.name})` : `(${user.name})`,
                        value: user.id,
                    })
                  )
                : resultRelatives.value?.userPublics.data?.filter(user => user.id == me?.id)?.map(user =>
                    ({
                        label: user.nickname ? `${user.nickname} (${user.name})` : `(${user.name})`,
                        value: user.id,
                    })
                  )
});
const dataUserSelect = ref(me?.id);
if (hasPermissionToReadUser && query.user_id) {
    dataUserSelect.value = query.user_id
}
const filterOptionUser = (input: any, option: any) => {
    return option.label.toLowerCase().indexOf(input.toLowerCase()) >= 0;
};

const dataTimeRange = ref(query.time_title ? query.time_title : props.initDateValue[0]);
const calendarValue = ref(query.time_start && query.time_end
    ? [dayjs(query.time_start), dayjs(query.time_end)]
    : props.initDateValue[1]
);
const customValue = ref(query.time_start && query.time_end
    ? [dayjs(query.time_start), dayjs(query.time_end)]
    : props.initDateValue[1]
);
const dates = ref();
const variables = computed(() => {
  let variableTemp = {};
  if(dataTimeRange.value !== 'custom' && calendarValue.value) {
    variableTemp = {
      dateRange: {
          from: calendarValue.value.includes(null) ? calendarValue?.value[0] : dayjs(calendarValue.value[0]).format('YYYY-MM-DD'),
          to: calendarValue.value.includes(null) ? calendarValue?.value[1] : dayjs(calendarValue.value[1]).format('YYYY-MM-DD'),
      }
    }
  } else if(dataTimeRange.value === 'custom' && customValue.value) {
    variableTemp = {
        dateRange: {
            from: customValue.value.includes(null) ? customValue?.value[0] : dayjs(customValue.value[0]).format('YYYY-MM-DD'),
            to: customValue.value.includes(null) ? customValue?.value[1] : dayjs(customValue.value[1]).format('YYYY-MM-DD'),
        }
    }
  }
  if(props.hasUserFilter) {
      variableTemp['userId'] = ['', null, undefined].includes(dataUserSelect.value) ? null : dataUserSelect.value;
  }
  return variableTemp
});

const routerObject = computed(() => {
    let queryTemp = {};
    if(calendarValue.value) {
        queryTemp = {
            path: props.initPath ?? location.pathname,
            query: {
                time_title: dataTimeRange?.value,
                time_start: calendarValue.value.includes(null) ? calendarValue?.value[0] : dayjs(calendarValue.value[0]).format('YYYY-MM-DD'),
                time_end: calendarValue.value.includes(null) ? calendarValue?.value[1] : dayjs(calendarValue.value[1]).format('YYYY-MM-DD'),
            }
        }
    } else if(dataTimeRange.value === 'custom' && customValue.value) {
        queryTemp = {
            path: props.initPath ?? location.pathname,
            query: {
                time_title: dataTimeRange?.value,
                time_start: customValue.value.includes(null) ? customValue?.value[0] : dayjs(customValue.value[0]).format('YYYY-MM-DD'),
                time_end: customValue.value.includes(null) ? customValue?.value[1] : dayjs(customValue.value[1]).format('YYYY-MM-DD'),
            }
        }
    }
    if(props.hasUserFilter) {
        queryTemp.query['userId'] = dataUserSelect.value
    }
    return queryTemp
});

const refetchData = computed(() => {
    return {
        variables: variables.value,
        routerObject: routerObject.value,

        userId: dataUserSelect.value,
        selectTimeQuickly: dataTimeRange.value,
        timeRange: dataTimeRange.value === 'custom' ? customValue.value : calendarValue.value,
    }
});
const emitRefetchData = () => {
  emit('refetchData', refetchData.value);
};

const handleChangeUser = (user: string) => {
  dataUserSelect.value = user;
  emitRefetchData();
};
const handleSelectTimeRange = () => {
  if(dataTimeRange.value === 'today') {
      calendarValue.value = [
        dayjs(),
        dayjs()
      ];
  } else if(dataTimeRange.value === 'yesterday') {
      calendarValue.value = [
        dayjs().subtract(1, 'day'),
        dayjs().subtract(1, 'day')
      ];
  } else if(dataTimeRange.value === 'this_week') {
      calendarValue.value = [
        dayjs().startOf('week'),
        dayjs().endOf('week')
      ];
  } else if(dataTimeRange.value === 'last_7_days') {
      calendarValue.value = [
        dayjs().subtract(7, 'day'),
        dayjs().subtract(1, 'day')
      ];
  } else if(dataTimeRange.value === 'last_week') {
      calendarValue.value = [
        dayjs().subtract(1, 'week').startOf('week'),
        dayjs().subtract(1, 'week').endOf('week')
      ];
  } else if(dataTimeRange.value === 'last_14_days') {
      calendarValue.value = [
        dayjs().subtract(14, 'day'),
        dayjs().subtract(1, 'day')
      ];
  } else if(dataTimeRange.value === 'this_month') {
      calendarValue.value = [
        dayjs().startOf('month'),
        dayjs().endOf('month')
      ];
  }  else if(dataTimeRange.value === 'last_30_days') {
      calendarValue.value = [
          dayjs().subtract(30, 'day'),
          dayjs().subtract(1, 'day')
      ];
  }   else if(dataTimeRange.value === 'last_month') {
      calendarValue.value = [
        dayjs().subtract(1, 'month').startOf('month'),
        dayjs().subtract(1, 'month').endOf('month')
      ];
  } else if(dataTimeRange.value === 'this_year') {
      calendarValue.value = [
        dayjs().startOf('year'),
        dayjs().endOf('year')
      ];
  } else if(dataTimeRange.value === 'all_time') {
      calendarValue.value = [null, null];
  }
  if(dataTimeRange.value !== 'custom') {
      emitRefetchData();
  }
  console.log('refetchData', refetchData.value)
};
const handleRangePickerChange = (val: any) => {
  customValue.value = val;
  emitRefetchData();
  console.log('refetchData custom', refetchData.value)
};
const disabledDate = current => {
    if (!dates.value || dates.value?.length === 0) {
        return false;
    }
    if(props.hasDisbleDate) {
      // const tooLate = dates.value[0] && current.diff(dates.value[0], 'days') < 6;
      const tooEarly = dates.value[1] && dates.value[1].diff(current, 'days') <= 7;
      return tooEarly;
    }
};
const handleRangePickerOpenChange = (open) => {
    if (open) {
        dates.value = [];
        calendarValue.value = [];
    } else {
        calendarValue.value = undefined;
    }
};
const handleRangePickerCalendarChange = (val) => {
    dates.value = val;
};

const spaceDirection = ref();
function useBreakpoints() {
    let windowWidth = ref(window.innerWidth);
    const onWidthChange = () => (windowWidth.value = window.innerWidth);
    onMounted(() => window.addEventListener('resize', onWidthChange));
    onUnmounted(() => window.removeEventListener('resize', onWidthChange));
    const width = computed(() => windowWidth.value);
    return { width };
}
const { width } = useBreakpoints();
onMounted(() => {
    if(width.value >= props.windowWidth) {
        spaceDirection.value = 'horizontal';
    } else {
        spaceDirection.value = 'vertical';
    }
});
watch(width, () => {
    if(width.value >= props.windowWidth) {
        spaceDirection.value = 'horizontal';
    } else {
        spaceDirection.value = 'vertical';
    }
});
</script>

<style lang="scss">
.extra-filter-wrapper {
    width: 100%;

    .button-first, .button-second, .select-user-cover, .select-time-title-cover {
        margin-bottom: 12px;
    }
    .select-user {
        min-width: 175px;
    }
    .select-time-title {
        min-width: 99px;
    }
    .range-picker {
        min-width: 220px;
    }

    @media (max-width: 768px) {
        .ant-btn {
          .text {
            display: none;
          }
          .icon {
            margin-right: 0;
          }
        }
    }
}
</style>
