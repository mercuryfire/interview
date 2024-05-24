<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn
          color="primary"
          class="q-mt-md"
          @click="handleCreateBtn(tempData)"
        >新增</q-btn>
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
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps, useQuasar } from 'quasar';
import { ref, onMounted } from 'vue';

const $q = useQuasar();

interface btnType {
  label: string;
  icon: string;
  status: string;
}
interface dataType {
  name: string;
  age: string;
}
interface rowDataType {
  id: string;
  name: string;
  age: string;
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

async function handleCreateBtn(data: dataType) {
  try {
    const res = await axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', {
      name: data.name,
      age: data.age,
    }, {
      headers: {
        'Content-Type': 'application/json',
      },
    });

    if (res.status === 200) {
      // Clear input
      tempData.value = {
        name: '',
        age: '',
      };

      // Refresh table data
      getTableData();

      console.log('新增成功');
    } else {
      console.log('新增失敗');
    }
  } catch (error) {
    console.error(error);
  }
}

async function deleteData(id: string) {
  try {
    const res = await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${id}`);
    if (res.status === 200) {
      console.log('刪除成功');
    } else {
      console.log(res)
      console.log('刪除失敗');
    }

    // Refresh table data
    getTableData();
  } catch (error) {
    console.error(error);
  }
}

async function getTableData() {
  try {
    const res = await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a');
    if (res.status === 200) {
      blockData.value = res.data;
    } else {
      console.log('取得資料失敗');
    }
  } catch (error) {
    console.error(error);
  }
}

function handleClickOption(btn: btnType, rowData: rowDataType) {
  if (btn.status === 'edit') {
  } else if (btn.status === 'delete') {
    $q.dialog({
      title: '提示',
      message: '是否確定刪除該筆資料?',
      ok: {
        label: '確定',
        flat: true,
        color: 'brown'
      },
      cancel: {
        label: '取消',
        flat: true,
        color: 'brown'
      },
    }).onOk(async () => {
      await deleteData(rowData.id);
    })
  }
}

onMounted(() => {
  getTableData();
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
