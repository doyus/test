<template>
  <BasicDrawer
    v-bind="$attrs"
    @register="registerDrawer"
    showFooter
    :title="getTitle"
    width="50%"
    @ok="handleSubmit"
  >
    <BasicForm @register="registerForm" />
  </BasicDrawer>
</template>
<script lang="ts">
  import { defineComponent, ref, computed, unref } from 'vue';
  import { BasicForm, useForm } from '/@/components/Form/index';
  import { BasicDrawer, useDrawerInner } from '/@/components/Drawer';
  import { createOrUpdate } from './api';
  import { formSchema } from './data';
  import { useI18n } from '/@/hooks/web/useI18n';

  export default defineComponent({
    name: 'ButtonDrawer',
    components: { BasicDrawer, BasicForm },
    emits: ['success', 'register'],
    setup(_, { emit }) {
      const isUpdate = ref(true);
      const { t } = useI18n();

      const [registerForm, { resetFields, setFieldsValue, validate }] = useForm({
        labelWidth: 100,
        schemas: formSchema,
        showActionButtonGroup: false,
        baseColProps: { lg: 12, md: 24 },
      });

      const [registerDrawer, { setDrawerProps, closeDrawer }] = useDrawerInner(async (data) => {
        await resetFields();
        setDrawerProps({ confirmLoading: false });
        isUpdate.value = !!data?.isUpdate;
        if (unref(isUpdate)) {
          await setFieldsValue({
            ...data.record,
            isUpdate: true,

          });
        }else {
          await setFieldsValue({
            isUpdate: false,
            demand_side: '元博网', // 只在添加新记录时设置默认值
          });}
      });

      const getTitle = computed(() =>
        !unref(isUpdate) ? t('common.addText') : t('common.updateText'),
      );

      async function handleSubmit() {
        try {
          const values = await validate();
          console.log("3333", values);
          // const { isUpdate1, ...submitValues } = values;
          // console.log("2222", isUpdate1);
          setDrawerProps({ confirmLoading: true });
          await createOrUpdate(values, unref(isUpdate));
          closeDrawer();
          emit('success');
        } finally {
          setDrawerProps({ confirmLoading: false });
        }
      }

      return {
        registerDrawer,
        registerForm,
        getTitle,
        handleSubmit,
      };
    },
  });
</script>
