<template>
  <div class="about">
    <h1 :style="{ fontSize: '32px', marginTop: '0', marginBottom: '5px' }">
      {{ questionDetail?.title }}
    </h1>
    <el-tabs v-model="activeName" class="demo-tabs" @tab-click="handleClick">
      <el-tab-pane label="题目描述" name="first">
        <div>
          <el-row :gutter="20">
            <el-col :span="12" :xs="24">
              <el-card class="box-card">
                <template #header>
                  <div class="card-header">
                    <span>题目内容</span>
                    <el-tag
                      class="ml-2"
                      type="success"
                      v-for="(item, index) in questionDetail?.tags"
                      :key="index"
                      >{{ item }}</el-tag
                    >
                  </div>
                </template>
                <MdViewer :value="questionDetail?.content"></MdViewer> </el-card
            ></el-col>
            <el-col :span="12" :xs="24">
              <div class="language">
                编程语言
                <el-select
                  v-model="form.language"
                  class="m-2"
                  placeholder="Select"
                  size="large"
                >
                  <el-option value="java" />
                  <el-option value="cpp" />
                  <el-option value="go" />
                  <el-option value="html" />
                </el-select>
              </div>

              <CodeEditor
                :value="form.code"
                :language="form.language"
                :handle-change="onCodeChange"
              >
              </CodeEditor>
              <el-button
                type="primary"
                size="large"
                style="margin-top: 10px"
                @click="doSubmit"
                v-loading="loading"
                >提交代码</el-button
              >
            </el-col>
          </el-row>
        </div>
      </el-tab-pane>
      <el-tab-pane label="题解" name="key">
        <div class="split-container">
          <el-card
            style="width: 100vw"
            @click="table = true"
            v-for="o in 2"
            :key="o"
          >
            <template #header>
              <div class="card-header">
                <span>题解标题</span>
              </div>
            </template>
            <p v-for="o in 2" :key="o" class="text item">
              {{ "简介" + o }}
            </p>
            <template #footer>点赞数，收藏数，评论数</template>
          </el-card>

          <el-drawer
            v-model="table"
            :title="questionKey.title"
            direction="rtl"
            size="50%"
          >
            <MdViewer :value="questionKey.content"></MdViewer>
          </el-drawer>

          <!-- <div class="left-panel" :style="{ width: leftWidth + 'px' }">
            左侧页面内容
          </div> -->

          <!-- <div
            class="separator"
            @mousedown="startDrag"
            @mouseup="stopDrag"
            @mousemove="doDrag"
          >
            <button class="drag-handle"></button>
          </div> -->
          <!-- <div class="right-panel" :style="{ width: rightWidth + 'px' }">
            右侧页面内容
          </div> -->
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>
<script setup lang="ts">
import { ref } from "vue";
import { ElMessage, type TabsPaneContext } from "element-plus";
import CodeEditor from "@/components/CodeEditor.vue";
import MdViewer from "@/components/MdViewer.vue";
import { useRoute } from "vue-router";
import {
  QuestionControllerService,
  QuestionSubmitAddRequest,
  QuestionVO,
} from "../../generated";
import router from "@/router";
const route = useRoute();

//加载
const loading = ref<boolean>(false);

//题目详情信息
const questionDetail = ref<QuestionVO>();
const questionId = route.params.id;
const getQuestionDetail = async () => {
  const res = await QuestionControllerService.getQuestionVoByIdUsingGet(
    questionId
  );
  if (res.code !== 0) {
    ElMessage.error("获取题目详情失败" + res.message);
    router.push("/");
    return;
  }
  console.log(res);
  questionDetail.value = res.data;
};
getQuestionDetail();
const activeName = ref("first");

const handleClick = (tab: TabsPaneContext, event: Event) => {
  console.log(tab, event);
};

// 代码编辑器
const form = ref<QuestionSubmitAddRequest>({
  code: "",
  language: "java",
  questionId: questionId,
});
const onCodeChange = (v: string) => {
  form.value.code = v;
};

const doSubmit = async () => {
  loading.value = true;
  const res = await QuestionControllerService.doQuestionSubmitUsingPost(
    form.value
  );
  if (res.code !== 0) {
    ElMessage.error("提交失败" + res.message);
    loading.value = false;
    return;
  }
  ElMessage.success("提交成功");
  setTimeout(() => {
    loading.value = false;
    router.push("/questionSubmit");
  }, 2000);
};

//题解页面
import { ElDrawer } from "element-plus";

const table = ref(false);
const questionKey = {
  title: "小胖爱喝汽水的题解",
  content:
    "## 小胖爱喝汽水的题解\n" +
    "\n" +
    "### 思路\n" +
    "\n" +
    "**使用数位排序**\n" +
    "\n" +
    "### 代码\n" +
    "\n" +
    "```java\n" +
    "import java.util.*;\n" +
    "public class Main{\n" +
    "    public static void main(String[]args){\n" +
    "        Scanner e = new Scanner(System.in);\n" +
    "        int n = e.nextInt();\n" +
    "        int x = e.nextInt();\n" +
    "        Integer a[] = new Integer[n + 1];\n" +
    "        int b[] = new int[n + 1];//预处理每个数对应的数位值\n" +
    "        for(int i = 0;i<=n;i++){\n" +
    "            a[i] = i;\n" +
    "            int z = 0;\n" +
    "            for(int j = i;j!=0;j /= 10) \n" +
    "                z += j%10;\n" +
    "                \n" +
    "            b[i] = z;\n" +
    "        } \n" +
    "        Arrays.sort(a,(q,w)->{return b[q] == b[w] ? q - w : b[q] - b[w];});//使用系统快排比较\n" +
    "        System.out.println(a[x]);\n" +
    "    }\n" +
    "}\n" +
    "```\n" +
    "\n",
};
const gridData = [
  {
    date: "2016-05-02",
    name: "Peter Parker",
    address: "Queens, New York City",
  },
  {
    date: "2016-05-04",
    name: "Peter Parker",
    address: "Queens, New York City",
  },
  {
    date: "2016-05-01",
    name: "Peter Parker",
    address: "Queens, New York City",
  },
  {
    date: "2016-05-03",
    name: "Peter Parker",
    address: "Queens, New York City",
  },
];

const isDragging = ref(false);
const startX = ref(0);
// 在 setup 函数中设置初始的 leftWidth 和 rightWidth
const leftWidth = ref(window.innerWidth * 0.45); // 左侧宽度占45%
const rightWidth = ref(window.innerWidth * 0.55); // 右侧宽度占55%

const startDrag = (e: MouseEvent) => {
  isDragging.value = true;
  startX.value = e.clientX;
};

const stopDrag = () => {
  isDragging.value = false;
};

const doDrag = (e: MouseEvent) => {
  if (isDragging.value) {
    const offset = e.clientX - startX.value;
    leftWidth.value += offset;
    rightWidth.value -= offset;
    startX.value = e.clientX;
  }
};
</script>
<style scoped lang="scss">
.card-header {
  display: flex;
  justify-content: space-between;
}
.language {
  margin-bottom: 5px;
}

//题解

.separator {
  position: relative;
  height: 100%;
  user-select: none; /* 防止选中文本 */
  touch-action: none; /* 禁止触摸事件的默认行为 */
  will-change: width; /* 告诉浏览器该元素的 width 属性将会发生改变 */
  transition: width 0.2s ease; /* 添加过渡效果 */
}
.drag-handle {
  position: absolute;
  left: -7.5px; /* 将按钮左移一半长度，使其居中 */
  width: 15px;
  height: 50px;
  cursor: col-resize;
  background-color: #ddd;
  border: none;
}
</style>
