<template>
    <tab
        :label="$gettext('Basic Info')"
        :item-header-class="tabClass"
    >
        <div class="row g-3">
            <form-group-multi-check
                id="form_edit_adapter"
                class="col-md-12"
                :field="v$.adapter"
                :options="adapterOptions"
                stacked
                radio
                :label="$gettext('Storage Adapter')"
            />

            <form-group-field
                id="form_edit_path"
                class="col-md-12"
                :field="v$.path"
                :label="$gettext('Path/Suffix')"
                :description="$gettext('For local filesystems, this is the base path of the directory. For remote filesystems, this is the folder prefix.')"
            />

            <form-group-field
                id="form_edit_storageQuota"
                class="col-md-12"
                :field="v$.storageQuota"
            >
                <template #label>
                    {{ $gettext('Storage Quota') }}
                </template>
                <template #description>
                    {{
                        $gettext('Set a maximum disk space that this storage location can use. Specify the size with unit, i.e. "8 GB". Units are measured in 1024 bytes. Leave blank to default to the available space on the disk.')
                    }}
                </template>
            </form-group-field>
        </div>
    </tab>
</template>

<script setup lang="ts">
import FormGroupField from "~/components/Form/FormGroupField.vue";
import {computed} from "vue";
import FormGroupMultiCheck from "~/components/Form/FormGroupMultiCheck.vue";
import {useTranslate} from "~/vendor/gettext";
import {useVuelidateOnFormTab} from "~/functions/useVuelidateOnFormTab";
import {required} from "@vuelidate/validators";
import Tab from "~/components/Common/Tab.vue";
import {ApiGenericForm, StorageLocationAdapters} from "~/entities/ApiInterfaces.ts";

const form = defineModel<ApiGenericForm>('form', {required: true});

const {v$, tabClass} = useVuelidateOnFormTab(
    form,
    {
        adapter: {required},
        path: {},
        storageQuota: {},
    },
    {
        adapter: StorageLocationAdapters.Local,
        path: '',
        storageQuota: '',
    }
);

const {$gettext} = useTranslate();

const adapterOptions = computed(() => {
    return [
        {
            value: StorageLocationAdapters.Local,
            text: $gettext('Local Filesystem')
        },
        {
            value: StorageLocationAdapters.S3,
            text: $gettext('Remote: S3 Compatible')
        },
        {
            value: StorageLocationAdapters.Dropbox,
            text: $gettext('Remote: Dropbox')
        },
        {
            value: StorageLocationAdapters.Sftp,
            text: $gettext('Remote: SFTP')
        }
    ];
});

</script>
