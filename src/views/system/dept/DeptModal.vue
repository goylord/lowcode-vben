<template>
  <BasicModal v-bind="$attrs" @register="registerModal" :title="getTitle" @ok="handleSubmit">
    <BasicForm @register="registerForm" />
  </BasicModal>
</template>
<script lang="ts">
  import { defineComponent, ref, computed, unref } from 'vue';
  import { BasicModal, useModalInner } from '/@/components/Modal';
  import { BasicForm, useForm } from '/@/components/Form/index';
  import { formSchema } from './dept.data';

  import { getDeptListTree, AddDept, EditDept } from '/@/api/demo/system';
  export default defineComponent({
    name: 'DeptModal',
    components: { BasicModal, BasicForm },
    emits: ['success', 'register'],
    setup(_, { emit }) {
      const isUpdate = ref(true);
      const rowId = ref('');

      const [registerForm, { resetFields, setFieldsValue, updateSchema, validate }] = useForm({
        labelWidth: 100,
        schemas: formSchema,
        showActionButtonGroup: false,
      });

      const [registerModal, { setModalProps, closeModal }] = useModalInner(async (data) => {
        resetFields();
        setModalProps({ confirmLoading: false });
        isUpdate.value = !!data?.isUpdate;

        if (unref(isUpdate)) {
          rowId.value = data?.record?.id;
          setFieldsValue({
            ...data.record,
          });
        }
        let treeData: any = await getDeptListTree();
        console.log('获取树的结果', treeData?.items);
        updateSchema({
          field: 'parent_dept',
          componentProps: { treeData: treeData?.items || [] },
        });
      });

      const getTitle = computed(() => (!unref(isUpdate) ? '新增部门' : '编辑部门'));

      async function handleSubmit() {
        try {
          const values = await validate();
          setModalProps({ confirmLoading: true });
          // TODO custom api
          console.log(values);
          if (isUpdate.value) {
            const result = await EditDept({
              id: rowId.value,
              ...values,
            });
            console.log(result);
            emit('success', { isUpdate: unref(isUpdate), values: { ...values, id: rowId.value } });
            closeModal();
          } else {
            // 新增
            const result = await AddDept(values);
            emit('success', { isUpdate: unref(isUpdate), values: { ...values, id: rowId.value } });
            console.log(result);
            closeModal();
          }
        } finally {
          setModalProps({ confirmLoading: false });
        }
      }

      return { registerModal, registerForm, getTitle, handleSubmit };
    },
  });
</script>
