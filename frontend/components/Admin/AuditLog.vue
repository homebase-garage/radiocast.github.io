<template>
    <card-page header-id="hdr_audit_log">
        <template #header="{id}">
            <div class="d-flex align-items-center">
                <h2
                    :id="id"
                    class="card-title flex-fill my-0"
                >
                    {{ $gettext('Audit Log') }}
                </h2>
                <div class="flex-shrink">
                    <date-range-dropdown
                        v-model="dateRange"
                        class="btn-dark"
                    />
                </div>
            </div>
        </template>

        <data-table
            paginated
            :fields="fields"
            :provider="apiItemProvider"
        >
            <template #cell(operation)="row">
                <span
                    v-if="row.item.operationText === 'insert'"
                    class="text-success"
                    :title="$gettext('Insert')"
                >
                    <icon
                        class="lg inline"
                        :icon="IconAddCircle"
                    />
                </span>
                <span
                    v-else-if="row.item.operationText === 'delete'"
                    class="text-danger"
                    :title="$gettext('Delete')"
                >
                    <icon
                        class="lg inline"
                        :icon="IconRemoveCircle"
                    />
                </span>
                <span
                    v-else
                    class="text-primary"
                    :title="$gettext('Update')"
                >
                    <icon
                        class="lg inline"
                        :icon="IconSwapHorizontalCircle"
                    />
                </span>
            </template>
            <template #cell(identifier)="row">
                <small>{{ row.item.class }}</small><br>
                {{ row.item.identifier }}
            </template>
            <template #cell(target)="row">
                <template v-if="row.item.target">
                    <small>{{ row.item.targetClass }}</small><br>
                    {{ row.item.target }}
                </template>
                <template v-else>
                    {{ $gettext('N/A') }}
                </template>
            </template>
            <template #cell(actions)="row">
                <template v-if="row.item.changes.length > 0">
                    <button
                        type="button"
                        class="btn btn-sm btn-primary"
                        @click="showDetails(row.item.changes)"
                    >
                        {{ $gettext('Changes') }}
                    </button>
                </template>
            </template>
        </data-table>
    </card-page>

    <details-modal ref="$detailsModal" />
</template>

<script setup lang="ts">
import {computed, ref, useTemplateRef} from "vue";
import {useTranslate} from "~/vendor/gettext";
import {useAzuraCast} from "~/vendor/azuracast";
import DataTable, {DataTableField} from "~/components/Common/DataTable.vue";
import DateRangeDropdown from "~/components/Common/DateRangeDropdown.vue";
import Icon from "~/components/Common/Icon.vue";
import DetailsModal from "~/components/Admin/AuditLog/DetailsModal.vue";
import CardPage from "~/components/Common/CardPage.vue";
import {useLuxon} from "~/vendor/luxon";
import {getApiUrl} from "~/router";
import {IconAddCircle, IconRemoveCircle, IconSwapHorizontalCircle} from "~/components/Common/icons";
import {ApiAdminAuditLogChangeset, AuditLog} from "~/entities/ApiInterfaces.ts";
import {DeepRequired} from "utility-types";
import {useApiItemProvider} from "~/functions/dataTable/useApiItemProvider.ts";
import {QueryKeys} from "~/entities/Queries.ts";

const baseApiUrl = getApiUrl('/admin/auditlog');

const {DateTime} = useLuxon();

const dateRange = ref({
    startDate: DateTime.now().minus({days: 13}).toJSDate(),
    endDate: DateTime.now().toJSDate(),
});

const {$gettext} = useTranslate();
const {timeConfig} = useAzuraCast();

type Row = AuditLog;

const fields: DataTableField<Row>[] = [
    {
        key: 'timestamp',
        label: $gettext('Date/Time'),
        sortable: false,
        formatter: (value) => {
            return DateTime.fromISO(value).toLocaleString(
                {
                    ...DateTime.DATETIME_SHORT, ...timeConfig
                }
            );
        }
    },
    {key: 'user', label: $gettext('User'), sortable: false},
    {key: 'operation', isRowHeader: true, label: $gettext('Operation'), sortable: false},
    {key: 'identifier', label: $gettext('Identifier'), sortable: false},
    {key: 'target', label: $gettext('Target'), sortable: false},
    {key: 'actions', label: $gettext('Actions'), sortable: false}
];

const apiUrl = computed(() => {
    const apiUrl = new URL(baseApiUrl.value, document.location.href);

    const apiUrlParams = apiUrl.searchParams;
    apiUrlParams.set('start', DateTime.fromJSDate(dateRange.value.startDate).toISO());
    apiUrlParams.set('end', DateTime.fromJSDate(dateRange.value.endDate).toISO());

    return apiUrl.toString();
});

const apiItemProvider = useApiItemProvider<Row>(
    apiUrl,
    [
        QueryKeys.AdminAuditLog,
        dateRange
    ]
);

const $detailsModal = useTemplateRef('$detailsModal');

type AuditLogChanges = DeepRequired<ApiAdminAuditLogChangeset>

const showDetails = (changes: AuditLogChanges[]) => {
    $detailsModal.value?.open(changes);
}
</script>
