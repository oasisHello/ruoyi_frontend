<template>
  <div class="app-container">


    <el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
    <!--View:Search condition area start-->
      <el-form-item label="name" prop="courseName">
        <el-input
          v-model="queryParams.courseName"
          placeholder="请输入name"
          clearable 
          @keyup.enter="handleQuery" 
        />

      </el-form-item>
      <el-form-item label="code" prop="courseCode">
        <el-input
          v-model="queryParams.courseCode"
          placeholder="请输入code"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="instructor" prop="instructor">
        <el-input
          v-model="queryParams.instructor"
          placeholder="请输入instructor"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="startdate" prop="startDate">
        <el-date-picker clearable
          v-model="queryParams.startDate"
          type="date"
          value-format="YYYY-MM-DD"
          placeholder="请选择startdate">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="enddate" prop="endDate">
        <el-date-picker clearable
          v-model="queryParams.endDate"
          type="date"
          value-format="YYYY-MM-DD"
          placeholder="请选择enddate">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="credits" prop="credits">
        <el-input
          v-model="queryParams.credits"
          placeholder="请输入credits"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="department" prop="department">
        <el-select v-model="queryParams.department" placeholder="请选择department" clearable>
          <el-option
            v-for="dict in department"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="created_at" prop="createdAt">
        <el-date-picker clearable
          v-model="queryParams.createdAt"
          type="date"
          value-format="YYYY-MM-DD"
          placeholder="请选择created_at">
        </el-date-picker>
      </el-form-item>
      <!--Search and reset Button-->
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>
    <!--View:Search condition area end-->
<!-- Button area-->
    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="Plus"
          @click="handleAdd"
          v-hasPermi="['course:course:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['course:course:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['course:course:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"
          @click="handleExport"
          v-hasPermi="['course:course:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>
<!--data table sheet start-->
    <el-table v-loading="loading" :data="courseList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="id" align="center" prop="courseId" />
      <el-table-column label="name" align="center" prop="courseName" />
      <el-table-column label="code" align="center" prop="courseCode" />
      <el-table-column label="description" align="center" prop="description" />
      <el-table-column label="instructor" align="center" prop="instructor" />
      <el-table-column label="startdate" align="center" prop="startDate" width="180">
        <template #default="scope">
          <span>{{ parseTime(scope.row.startDate, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="enddate" align="center" prop="endDate" width="180">
        <template #default="scope">
          <span>{{ parseTime(scope.row.endDate, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="credits" align="center" prop="credits" />
      <el-table-column label="department" align="center" prop="department">
        <template #default="scope">
          <dict-tag :options="department" :value="scope.row.department"/>
        </template>
      </el-table-column>
      <el-table-column label="created_at" align="center" prop="createdAt" width="180">
        <template #default="scope">
          <span>{{ parseTime(scope.row.createdAt, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)" v-hasPermi="['course:course:edit']">修改</el-button>
          <el-button link type="primary" icon="Delete" @click="handleDelete(scope.row)" v-hasPermi="['course:course:remove']">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!--data table sheet end-->
    
    <!--pagination-->
    <pagination
      v-show="total>0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改course对话框 -->
    <el-dialog :title="title" v-model="open" width="500px" append-to-body>
      <el-form ref="courseRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="name" prop="courseName">
          <el-input v-model="form.courseName" placeholder="请输入name" />
        </el-form-item>
        <el-form-item label="code" prop="courseCode">
          <el-input v-model="form.courseCode" placeholder="请输入code" />
        </el-form-item>
        <el-form-item label="description" prop="description">
          <el-input v-model="form.description" type="textarea" placeholder="请输入内容" />
        </el-form-item>
        <el-form-item label="instructor" prop="instructor">
          <el-input v-model="form.instructor" placeholder="请输入instructor" />
        </el-form-item>
        <el-form-item label="startdate" prop="startDate">
          <el-date-picker clearable
            v-model="form.startDate"
            type="date"
            value-format="YYYY-MM-DD"
            placeholder="请选择startdate">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="enddate" prop="endDate">
          <el-date-picker clearable
            v-model="form.endDate"
            type="date"
            value-format="YYYY-MM-DD"
            placeholder="请选择enddate">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="credits" prop="credits">
          <el-input v-model="form.credits" placeholder="请输入credits" />
        </el-form-item>
        <el-form-item label="department" prop="department">
          <el-select v-model="form.department" placeholder="请选择department">
            <el-option
              v-for="dict in department"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="created_at" prop="createdAt">
          <el-date-picker clearable
            v-model="form.createdAt"
            type="date"
            value-format="YYYY-MM-DD"
            placeholder="请选择created_at">
          </el-date-picker>
        </el-form-item>
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

<script setup name="Course">
// import the api of backended
import { listCourse, getCourse, delCourse, addCourse, updateCourse } from "@/api/course/course";

const { proxy } = getCurrentInstance();
const { department } = proxy.useDict('department');

const courseList = ref([]);
const open = ref(false);
// loading status
const loading = ref(true);
// 
const showSearch = ref(true);
const ids = ref([]);
const single = ref(true);
// 
const multiple = ref(true);
//number of records
const total = ref(0);
const title = ref("");

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    courseName: null,
    courseCode: null,
    description: null,
    instructor: null,
    startDate: null,
    endDate: null,
    credits: null,
    department: null,
    createdAt: null
  },
  rules: {
    courseName: [
      { required: true, message: "name不能为空", trigger: "blur" }
    ],
    courseCode: [
      { required: true, message: "code不能为空", trigger: "blur" }
    ],
  }
});

const { queryParams, form, rules } = toRefs(data);

/** 查询course列表 */
function getList() {
  loading.value = true;
  listCourse(queryParams.value).then(response => {
    courseList.value = response.rows;
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
    courseId: null,
    courseName: null,
    courseCode: null,
    description: null,
    instructor: null,
    startDate: null,
    endDate: null,
    credits: null,
    department: null,
    createdAt: null
  };
  proxy.resetForm("courseRef");
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
  ids.value = selection.map(item => item.courseId);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
}

/** 新增按钮操作 */
function handleAdd() {
  reset();
  open.value = true;
  title.value = "添加course";
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset();
  const _courseId = row.courseId || ids.value
  getCourse(_courseId).then(response => {
    form.value = response.data;
    open.value = true;
    title.value = "修改course";
  });
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["courseRef"].validate(valid => {
    if (valid) {
      if (form.value.courseId != null) {
        updateCourse(form.value).then(response => {
          proxy.$modal.msgSuccess("修改成功");
          open.value = false;
          getList();
        });
      } else {
        addCourse(form.value).then(response => {
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
  const _courseIds = row.courseId || ids.value;
  proxy.$modal.confirm('是否确认删除course编号为"' + _courseIds + '"的数据项？').then(function() {
    return delCourse(_courseIds);
  }).then(() => {
    getList();
    proxy.$modal.msgSuccess("删除成功");
  }).catch(() => {});
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download('course/course/export', {
    ...queryParams.value
  }, `course_${new Date().getTime()}.xlsx`)
}

getList();
</script>
