<template>
  <div id="shortAnswer">
    <el-form
      :model="question"
      :rules="rules"
      ref="question"
      label-width="100px"
      v-loading="formLoading"
    >
      <el-form-item label="题干：" prop="title" required>
        <RichEditor
          :id="0"
          :cont="question.title"
          @updateContent="updateContent"
        ></RichEditor>
      </el-form-item>

      <el-form-item label="答案：" prop="answer" required>
        <RichEditor
          :id="1"
          :cont="question.answer"
          @updateContent="updateContent"
        ></RichEditor>
      </el-form-item>

      <el-form-item label="解析：" prop="analyze" required>
        <RichEditor
          :id="2"
          :cont="question.analyze"
          @updateContent="updateContent"
        ></RichEditor>
      </el-form-item>

      <el-form-item label="分数：" prop="score" required>
        <el-input-number
          :min="0"
          :step="0.5"
          v-model="question.score"
        ></el-input-number>
      </el-form-item>

      <el-form-item label="难度：" prop="difficult" required>
        <el-rate
          v-model="question.difficult"
          class="question-item-rate"
        ></el-rate>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="submitQuestion">提交</el-button>
        <el-button @click="resetQuestion">重置</el-button>
        <el-button type="success" @click="questionVisible = true"
          >预览题目</el-button
        >
      </el-form-item>
    </el-form>
    <el-dialog
      :visible.sync="questionVisible"
      style="width: 100%; height: 100%"
    >
      <QuestionShow :question="question" />
    </el-dialog>
  </div>
</template>

<script>
import RichEditor from "@/components/RichEditor/index.vue";
import QuestionShow from "@/components/QuestionShow/index.vue";
export default {
  name: "trueFalse",
  components: {
    RichEditor,
    QuestionShow,
  },
  data() {
    return {
      isEdit: false,
      question: {
        id: null,
        questionType: 4,
        difficult: 0,
        title: "",
        answer: "",
        analyze: "",
        score: "",
      },
      formLoading: false,
      rules: {
        title: [{ required: true, message: "请输入题干", trigger: "blur" }],
        answer: [{ required: true, message: "请输入答案", trigger: "blur" }],
        analyze: [{ required: true, message: "请输入解析", trigger: "blur" }],
      },
      questionVisible: false,
    };
  },
  created() {
    let myRow = this.$route.params.question;
    if (myRow != null) {
      this.isEdit = true;

      this.question.id = myRow.qid;
      this.question.title = myRow.title;
      this.question.answer = myRow.answer;
      this.question.analyze = myRow.analysis;
      this.question.score = myRow.score;
      this.question.difficult = myRow.difficult;
      this.question.questionType = myRow.qtype;
    }
  },
  methods: {
    updateContent({ id, content }) {
      if (id === 0) {
        this.question.title = content;
      } else if (id === 1) {
        this.question.answer = content;
      } else if (id === 2) {
        this.question.analyze = content;
      }
    },
    submitQuestion() {
      if (!this.isEdit) {
        this.$refs.question.validate((valid) => {
          if (valid) {
            this.apis.question
              .submitQuestion(
                sessionStorage.getItem("teacherUsername"),
                sessionStorage.getItem("teacherName"),
                this.question.title,
                this.question.answer,
                this.question.analyze,
                "",
                this.question.score,
                this.question.difficult,
                4
              )
              .then((res) => {
                if (res.data.status === 200) {
                  this.$notify({
                    title: "成功",
                    message: "题目上传成功！",
                    type: "success",
                  });
                  this.$router.push({ name: "questionList" });
                }
              });
          } else {
            return false;
          }
        });
      } else {
        this.editQuestion();
      }
    },
    editQuestion() {
      this.$refs.question.validate((valid) => {
        if (valid) {
          this.apis.question
            .editQuestion(
              this.question.id,
              this.question.title,
              this.question.answer,
              this.question.analyze,
              "",
              this.question.score,
              this.question.difficult,
              4
            )
            .then((res) => {
              if (res.data.status === 200) {
                this.$notify({
                  title: "成功",
                  message: "题目修改成功！",
                  type: "success",
                });
                this.$router.push({ name: "questionList" });
              }
            });
        } else {
          return false;
        }
      });
    },
    resetQuestion() {
      let lastId = this.question.id;
      this.$refs.question.resetFields();
      this.question = {
        id: null,
        questionType: 3,
        difficult: 0,
        title: "",
        answer: "",
        analyze: "",
        score: "",
      };
    },
    showQuestion() {},
  },
};
</script>

<style scoped>
#shortAnswer {
  width: 100%;
  min-height: 50vh;
  box-sizing: border-box;
  padding: 20px;
}
</style>