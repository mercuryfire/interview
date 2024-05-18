<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input
          v-model="tempData.name"
          label="姓名"
          :rules="[(val) => !!val || '姓名不得空白']"
          lazy-rules
        />
        <q-input
          v-model="tempData.age"
          label="年齡"
          :rules="[
            (val) => !!val || '年齡不得空白',
            (val) => /^\d+$/.test(val) || '年齡必須是正整數',
          ]"
          lazy-rules
        />
        <q-btn color="primary" class="q-mt-md" @click="addData">新增</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>

      <q-dialog v-model="isDialogOpen">
        <q-card>
          <q-card-section>
            <div class="text-h6">編輯資料</div>
          </q-card-section>

          <q-card-section>
            <q-form @submit.prevent="saveEdit" ref="editFormRef">
              <q-input
                v-model="editData.name"
                label="姓名"
                :rules="[(val) => !!val || '姓名不得空白']"
                lazy-rules
              />
              <q-input
                v-model="editData.age"
                label="年齡"
                :rules="[
                  (val) => !!val || '年齡不得空白',
                  (val) => /^\d+$/.test(val) || '年齡必須是正整數',
                ]"
                lazy-rules
              />
              <q-card-actions align="right">
                <q-btn flat label="取消" v-close-popup />
                <q-btn type="submit" flat label="保存" />
              </q-card-actions>
            </q-form>
          </q-card-section>
        </q-card>
      </q-dialog>
      <q-dialog v-model="isDeleteDialogOpen">
        <q-card>
          <q-card-section>
            <div class="text-h6">提示</div>
          </q-card-section>

          <q-card-section> 是否確定刪除該筆資料？ </q-card-section>

          <q-card-actions align="right">
            <q-btn flat label="取消" v-close-popup />
            <q-btn flat label="刪除" color="red" @click="confirmDelete" />
          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { onMounted, ref } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: '',
  age: '',
});

const editData = ref<{ id: number; name: string; age: number }>({
  id: 0,
  name: '',
  age: 0,
});
const isDialogOpen = ref(false);
const isDeleteDialogOpen = ref(false);
const itemToDelete = ref<{ id: number; name: string; age: number } | null>(
  null
);
const formRef = ref();
const editFormRef = ref();
const handleClickOption = (btn, data) => {
  if (btn.status === 'delete') {
    // axios
    //   .delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${data.id}`)
    //   .then((response) => {
    //     blockData.value = blockData.value.filter((item) => item.id !== data.id);
    //   })
    //   .catch((error) => {
    //     console.error('Error deleting data:', error);
    //   });

    itemToDelete.value = data;
    isDeleteDialogOpen.value = true;
  } else if (btn.status === 'edit') {
    editData.value = { ...data };
    isDialogOpen.value = true;
  }
};

const confirmDelete = () => {
  axios
    .delete(
      `https://dahua.metcfire.com.tw/api/CRUDTest/${itemToDelete.value.id}`
    )
    .then((response) => {
      blockData.value = blockData.value.filter(
        (item) => item.id !== itemToDelete.value.id
      );
      isDeleteDialogOpen.value = false;
      itemToDelete.value = null;
    })
    .catch((error) => {
      console.error('Error deleting data:', error);
    });
};

const fetchData = () => {
  axios
    .get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
    .then((response) => {
      blockData.value = response.data;
    })
    .catch((error) => {
      console.error('Error fetching data:', error);
    });
};

const addData = () => {
  if (editFormRef.value && editFormRef.value.validate()) {
    axios
      .post('https://dahua.metcfire.com.tw/api/CRUDTest', tempData.value, {
        headers: { 'Content-Type': 'application/json' },
      })
      .then((response) => {
        console.log(response);
      })
      .catch((error) => {
        console.error('Error adding data:', error);
      });
  }
};

const saveEdit = () => {
  if (editFormRef.value && editFormRef.value.validate()) {
    axios
      .patch('https://dahua.metcfire.com.tw/api/CRUDTest', editData.value, {
        headers: { 'Content-Type': 'application/json' },
      })
      .then((response) => {
        const index = blockData.value.findIndex(
          (item) => item.id === editData.value.id
        );
        if (index !== -1) {
          blockData.value[index] = { ...editData.value };
        }
        isDialogOpen.value = false;
      })
      .catch((error) => {
        console.error('Error saving edit:', error);
      });
  }
};

onMounted(() => {
  fetchData();
});
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
