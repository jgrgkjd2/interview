<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-form @submit="insertData">
          <q-input v-model="tempData.name" label="姓名" />
          <q-input v-model="tempData.age" label="年齡" />
          <q-btn type="submit" color="primary" class="q-mt-md">{{ switchLabel }}</q-btn>
        </q-form>
      </div>

      <q-table flat bordered ref="tableRef" :rows="blockData" :columns="(tableConfig as QTableProps['columns'])"
        row-key="id" hide-pagination separator="cell" :rows-per-page-options="[0]">
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
            <q-td v-for="col in props.cols" :key="col.name" :props="props" style="min-width: 120px">
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn @click="handleClickOption(btn, props.row)" v-for="(btn, index) in tableButtons" :key="index"
                size="sm" color="grey-6" round dense :icon="btn.icon" class="q-ml-md" padding="5px 5px">
                <q-tooltip transition-show="scale" transition-hide="scale" anchor="top middle" self="bottom middle"
                  :offset="[10, 10]">
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div class="full-width row flex-center items-center text-primary q-gutter-sm" style="font-size: 18px">
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>

      <q-dialog v-model="noticeDialog" persistent>
        <q-card>
          <q-card-section class="q-pt-md q-pb-md">
            <q-card-title class="text-h6">{{ showMeesage }}</q-card-title>
          </q-card-section>

          <q-card-actions align="right" style="width: 300px;height: 50px;">
            <q-btn label="確定" color="primary" @click="noticeDialog = false" />
          </q-card-actions>
        </q-card>
      </q-dialog>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref, onMounted } from 'vue';

let noticeDialog = ref(false);
let showMeesage = ref("");
interface btnType {
  label: string;
  icon: string;
  status: string;
}
interface dataType {
  name: string;
  age: number;
  creatorId: string;
  id: string;
};
const switchLabel = ref("新增");
const blockData = ref([
  // {
  //   name: 'test',
  //   age: 25,
  // },
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
  age: '' as number | string,
});
const tempPost = ref({
  name: "",
  age: "" as number | string,
  creatorId: "",
  id: "",
})
async function handleClickOption(btn: btnType, data: dataType) {
  try {
    if (btn.status === "edit") {
      tempData.value = data;
      tempPost.value = data;
      switchLabel.value = "更新";
    }
    else {
      const res = await axios.delete(`https://demo.mercuryfire.com.tw:49110/crudTest/${data.id}`);
      console.log(`delete = ${res.data}`);
      getData();
      showMeesage.value = "刪除成功";
      noticeDialog.value = true;
    }
  }
  catch (error) {
    console.error(error);
  }
}
const getData = async () => {
  try {
    const res = await axios.get("https://demo.mercuryfire.com.tw:49110/crudTest/a");
    blockData.value = res.data.result;
    console.log(blockData.value);
  }
  catch (error) {
    console.error(`Get data error = ${error}`);
  }
};
const insertData = async () => {
  try {
    if (switchLabel.value === "更新") {
      tempData.value.age = Number(tempData.value.age);
      const res = await axios.post("https://demo.mercuryfire.com.tw:49110/crudTest", tempData.value);
      if (res) {
        await axios.delete(`https://demo.mercuryfire.com.tw:49110/crudTest/${tempPost.value.id}`);
      }
      getData();
      showMeesage.value = "修改成功";
      noticeDialog.value = true;
      tempData.value.age = "";
      tempData.value.name = "";
      switchLabel.value = "新增";
    }
    else {
      tempData.value.age = Number(tempData.value.age);
      await axios.post("https://demo.mercuryfire.com.tw:49110/crudTest", tempData.value);
      showMeesage.value = "新增成功";
      noticeDialog.value = true;
    }
  }
  catch (error) {
    console.error(`Insert data error = ${error}`);
  }
};
const checkInput = (data: dataType) => {
  // 沒引用
  if (data.age.toString.length === 0) {
    showMeesage.value = "數字未填";
    return false;
  }
  if (data.name.length === 0) {
    showMeesage.value = "名字未填";
    return false;
  }
  return true;
}
onMounted(() => {
  getData();
})
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
