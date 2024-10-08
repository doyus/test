<template>
  <div>
    <BasicTable @register="registerTable">
      <template #tableTitle>
        <Space style="height: 40px">
          <a-button
            type="primary"
            v-auth="['demo:add']"
            preIcon="ant-design:plus-outlined"
            @click="handleCreate"
          >
            {{ t('common.addText') }}
          </a-button>
          <a-button
            type="error"
            v-auth="['demo:delete']"
            preIcon="ant-design:delete-outlined"
            @click="handleBulkDelete"
          >
            {{ t('common.delText') }}
          </a-button>
          <!-- <BasicUpload
            :maxSize="20"
            :maxNumber="1"
            @change="handleChange"
            class="my-5"
            type="warning"
            :text="t('common.importText')"
            v-auth="['demo:update']"
          /> -->
          <!-- <a-button
            type="success"
            v-auth="['demo:update']"
            preIcon="carbon:cloud-download"
            @click="handleExportData"
          >
            {{ t('common.exportText') }}
          </a-button> -->
        </Space>
      </template>
      <template #bodyCell="{ column, record }">
        <template v-if="column.key === 'action'">
          <TableAction
          :actions="[
              {
                type: 'button',
                color: 'primary',
                icon: 'clarity:note-edit-line',
                onClick: handleEdit.bind(null, record),
                auth: ['menu:update'],
              },
              {
                type: 'button',
                color: 'error',
                icon: 'ant-design:delete-outlined',
                placement: 'left',
                auth: ['menu:delete'],
                popConfirm: {
                  title: t('common.delHintText'),
                  confirm: handleDelete.bind(null, record.id),
                },
              },
              {
                type: 'button',
                color: 'warning',
                tooltip: t('common.websiteInfo.addData'),
                icon: 'ant-design:plus-square-outlined',
                auth: ['menu:update'],
                onClick: handleCreate.bind(null, record),
              },
            ]"
          />
        </template>
      </template>
    </BasicTable>
    <DemoDrawer @register="registerDrawer" @success="handleSuccess" />
  </div>
</template>
<script lang="ts">
  import { defineComponent } from 'vue';
  import { useRouter } from 'vue-router';
  import { BasicTable, useTable, TableAction } from '/@/components/Table';
  import { usePermission } from '/@/hooks/web/usePermission';
  import { useDrawer } from '/@/components/Drawer';
  import DemoDrawer from './Drawer.vue';
  import { Space } from 'ant-design-vue';
  import { BasicUpload } from '/@/components/Upload';
  import { deleteItem, getList} from './api';
  import { columns, searchFormSchema } from './data';
  import { message } from 'ant-design-vue';
  import { useMessage } from '/@/hooks/web/useMessage';
  import { useI18n } from '/@/hooks/web/useI18n';
  export default defineComponent({
    name: 'Demo',
    components: { BasicTable, DemoDrawer, TableAction, BasicUpload, Space },
    setup() {
      const { t } = useI18n();

      const [registerDrawer, { openDrawer }] = useDrawer();
      const { createConfirm } = useMessage();
      const { hasPermission } = usePermission();
      const [registerTable, { reload, getSelectRows }] = useTable({
        api: getList,
        columns,
        formConfig: {
          labelWidth: 80,
          schemas: searchFormSchema,
        },
        useSearchForm: true,
        showTableSetting: true,
        tableSetting: { fullScreen: true },
        bordered: true,
        showIndexColumn: false,
        rowSelection: {
          type: 'checkbox',
        },
        // 去掉操作列
        // actionColumn: {
        //   width: 150,
        //   title: t('common.operationText'),
        //   dataIndex: 'action',
        //   fixed: undefined,
        // },
      });

      function handleCreate() {
        openDrawer(true, {
          isUpdate: false,
        });
      }

      function handleEdit(record: Recordable) {
        openDrawer(true, {
          record,
          isUpdate: true,
        });
      }

      async function handleDelete(id: number) {
        await deleteItem(id);
        message.success(t('common.successText'));
        await reload();
      }

      async function handleGoPage(arg: any) {
        const { router } = useRouter();
        router.push(pageEnum.BASE_LOGIN);
      }

      function handleBulkDelete() {
        if (getSelectRows().length == 0) {
          message.warning(t('common.batchDelHintText'));
        } else {
          createConfirm({
            iconType: 'warning',
            title: t('common.hintText'),
            content: t('common.delHintText'),
            async onOk() {
              for (const item of getSelectRows()) {
                await deleteItem(item.id);
              }
              message.success(t('common.successText'));
              await reload();
            },
          });
        }
      }

      function handleSuccess() {
        message.success(t('common.successText'));
        reload();
      }

      return {
        registerTable,
        registerDrawer,
        handleCreate,
        handleEdit,
        handleDelete,
        handleSuccess,
        hasPermission,
        handleBulkDelete,
        getSelectRows,
        t,
      };
    },
  });
</script>