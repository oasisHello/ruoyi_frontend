<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="DishName" prop="name">
        <el-input v-model="queryParams.name" placeholder="请输入DishName" clearable @keyup.enter="handleQuery" />
      </el-form-item>
      <el-form-item label="Status" prop="status">
        <el-select v-model="queryParams.status" placeholder="请选择Status" clearable>
          <el-option v-for="dict in dish_availability" :key="dict.value" :label="dict.label" :value="dict.value" />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button type="primary" plain icon="Plus" @click="handleAdd" v-hasPermi="['merchant:dish:add']">新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button type="success" plain icon="Edit" :disabled="single" @click="handleUpdate"
          v-hasPermi="['merchant:dish:edit']">修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button type="danger" plain icon="Delete" :disabled="multiple" @click="handleDelete"
          v-hasPermi="['merchant:dish:remove']">删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button type="warning" plain icon="Download" @click="handleExport"
          v-hasPermi="['merchant:dish:export']">导出</el-button>
      </el-col>
      <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="dishList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <!-- <el-table-column label="Primary Key" align="center" prop="id" /> -->
      <el-table-column label="DishName" align="center" prop="name" />
      <el-table-column label="Price" align="center">
        <template #default="scope">
          <div>
            ${{ scope.row.price }}
          </div>
        </template>
      </el-table-column>
      <el-table-column label="Image" align="center" prop="image" width="100">
        <template #default="scope">
          <image-preview :src="scope.row.image" :width="50" :height="50" />
        </template>
      </el-table-column>
      <el-table-column label="Status" align="center" prop="status">
        <template #default="scope">
          <dict-tag :options="dish_availability" :value="scope.row.status" />
        </template>
      </el-table-column>
      <el-table-column label="Update Time" align="center" prop="updateTime" width="180">
        <template #default="scope">
          <span>{{ parseTime(scope.row.updateTime, '{y}-{m}-{d} {h}:{i}:{s}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)"
            v-hasPermi="['merchant:dish:edit']">修改</el-button>
          <el-button link type="primary" icon="Delete" @click="handleDelete(scope.row)"
            v-hasPermi="['merchant:dish:remove']">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total > 0" :total="total" v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize" @pagination="getList" />

    <!-- 添加或修改merchant对话框 -->
    <el-dialog :title="title" v-model="open" width="600px" append-to-body>
      <el-form ref="dishRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="DishName" prop="name">
          <el-input v-model="form.name" placeholder="请输入DishName" />
        </el-form-item>
        <el-form-item label="Price" prop="price">
          <el-input v-model="form.price" placeholder="请输入Price" />
        </el-form-item>
        <el-form-item label="Image" prop="image">
          <image-upload v-model="form.image" />
        </el-form-item>
        <el-form-item label="Description" prop="description">
          <el-input v-model="form.description" placeholder="请输入Description" />
        </el-form-item>
        <el-form-item label="Status" prop="status">
          <el-select v-model="form.status" placeholder="请选择Status">
            <el-option v-for="dict in dish_availability" :key="dict.value" :label="dict.label"
              :value="parseInt(dict.value)"></el-option>
          </el-select>
        </el-form-item>
        <el-divider content-position="center">flavor信息</el-divider>
        <el-row :gutter="10" class="mb8">
          <el-col :span="1.5">
            <el-button type="primary" icon="Plus" @click="handleAddDishFlavor">添加</el-button>
          </el-col>
          <el-col :span="1.5">
            <el-button type="danger" icon="Delete" @click="handleDeleteDishFlavor">删除</el-button>
          </el-col>
        </el-row>
        <el-table :data="dishFlavorList" :row-class-name="rowDishFlavorIndex"
          @selection-change="handleDishFlavorSelectionChange" ref="dishFlavor">
          <el-table-column type="selection" width="50" align="center" />
          <el-table-column label="序号" align="center" prop="index" width="50" />
          <el-table-column label="flavor" prop="name" width="150">
            <template #default="scope">
              <el-select v-model="scope.row.name" placeholder="flavor" @change="changeFlavorValue(scope.row)">
                <el-option v-for="dishFlavor in dishFlavorCandidates" :key="dishFlavor.name" :label="dishFlavor.name"
                  :value="dishFlavor.name"></el-option>
              </el-select>
            </template>
          </el-table-column>
          <el-table-column label="flavor list" prop="value" width="300">
            <template #default="scope">
              <el-select v-model="scope.row.value" placeholder="favor values" multiple
              @focus="listFlavorValues(scope.row)" style="width:90%;">
                <el-option v-for="value in flavorValues" :key="value" :label="value" :value="value"></el-option>
              </el-select>
            </template>
          </el-table-column>
        </el-table>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup name="Dish">
import { listDish, getDish, delDish, addDish, updateDish } from "@/api/merchant/dish";

const { proxy } = getCurrentInstance();
const { dish_availability } = proxy.useDict('dish_availability');

const dishList = ref([]);
const dishFlavorList = ref([]);
const open = ref(false);
const loading = ref(true);
const showSearch = ref(true);
const ids = ref([]);
const checkedDishFlavor = ref([]);
const single = ref(true);
const multiple = ref(true);
const total = ref(0);
const title = ref("");

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    name: null,
    status: null,
  },
  rules: {
    name: [
      { required: true, message: "DishName不能为空", trigger: "blur" }
    ],
    price: [
      { required: true, message: "Price不能为空", trigger: "blur" }
    ],
    image: [
      { required: true, message: "Image不能为空", trigger: "blur" }
    ],
    status: [
      { required: true, message: "Status不能为空", trigger: "change" }
    ],
  }
});

const { queryParams, form, rules } = toRefs(data);

//---------------Custom section Start-----------
const dishFlavorCandidates = ref([
  { name: "spiciness", value: ["no spicy", "little spicy", "moderate spicy", "spicy"] },
  { name: "sweetness", value: ["no sweet", "little sweet", "moderate sweet", "sweet"] },
]);

// store the value of the selected flavor
const flavorValues = ref({});

// track the value of the selected flavor
function changeFlavorValue(row) {
  // clear
  row.value = [];

  // find the value by flavor name.
  flavorValues.value = dishFlavorCandidates.value.find(item => item.name === row.name).value;
}

// list all the flavor values　when flavor values is focused
function listFlavorValues(row) {
  // find the value by flavor name.
  flavorValues.value = dishFlavorCandidates.value.find(item => item.name === row.name).value;
}
//--------------Custom section End------------

/** 查询merchant列表 */
function getList() {
  loading.value = true;
  listDish(queryParams.value).then(response => {
    dishList.value = response.rows;
    total.value = response.total;
    loading.value = false;
  });
}

// 取消按钮
function cancel() {
  open.value = false;
  reset();
}

// 表单重置
function reset() {
  form.value = {
    id: null,
    name: null,
    price: null,
    image: null,
    description: null,
    status: null,
    createTime: null,
    updateTime: null
  };
  dishFlavorList.value = [];
  proxy.resetForm("dishRef");
}

/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1;
  getList();
}

/** 重置按钮操作 */
function resetQuery() {
  proxy.resetForm("queryRef");
  handleQuery();
}

// 多选框选中数据
function handleSelectionChange(selection) {
  ids.value = selection.map(item => item.id);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
}

/** 新增按钮操作 */
function handleAdd() {
  reset();
  open.value = true;
  title.value = "添加merchant";
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset();
  const _id = row.id || ids.value
  getDish(_id).then(response => {
    form.value = response.data;
    dishFlavorList.value = response.data.dishFlavorList;
    if (dishFlavorList.value != null) {
      dishFlavorList.value.forEach(item => {
        item.value = JSON.parse(item.value);
      });
    }

    open.value = true;
    title.value = "修改merchant";
  });
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["dishRef"].validate(valid => {
    if (valid) {
      form.value.dishFlavorList = dishFlavorList.value;
      form.value.dishFlavorList.forEach(item => {
        item.value = JSON.stringify(item.value);
      });
      if (form.value.id != null) {
        updateDish(form.value).then(response => {
          proxy.$modal.msgSuccess("修改成功");
          open.value = false;
          getList();
        });
      } else {
        addDish(form.value).then(response => {
          proxy.$modal.msgSuccess("新增成功");
          open.value = false;
          getList();
        });
      }
    }
  });
}

/** 删除按钮操作 */
function handleDelete(row) {
  const _ids = row.id || ids.value;
  proxy.$modal.confirm('是否确认删除merchant编号为"' + _ids + '"的数据项？').then(function () {
    return delDish(_ids);
  }).then(() => {
    getList();
    proxy.$modal.msgSuccess("删除成功");
  }).catch(() => { });
}

/** flavor序号 */
function rowDishFlavorIndex({ row, rowIndex }) {
  row.index = rowIndex + 1;
}

/** flavor添加按钮操作 */
function handleAddDishFlavor() {
  let obj = {};
  obj.name = "";
  obj.value = "";
  dishFlavorList.value.push(obj);
}

/** flavor删除按钮操作 */
function handleDeleteDishFlavor() {
  if (checkedDishFlavor.value.length == 0) {
    proxy.$modal.msgError("请先选择要删除的flavor数据");
  } else {
    const dishFlavors = dishFlavorList.value;
    const checkedDishFlavors = checkedDishFlavor.value;
    dishFlavorList.value = dishFlavors.filter(function (item) {
      return checkedDishFlavors.indexOf(item.index) == -1
    });
  }
}

/** 复选框选中数据 */
function handleDishFlavorSelectionChange(selection) {
  checkedDishFlavor.value = selection.map(item => item.index)
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download('merchant/dish/export', {
    ...queryParams.value
  }, `dish_${new Date().getTime()}.xlsx`)
}

getList();
</script>
