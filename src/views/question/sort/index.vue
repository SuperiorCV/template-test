<template>
  <div id="sort">
    <el-form
      :model="question"
      :rules="rules"
      ref="question"
      label-width="100px"
    >
      <el-form-item label="题干：" prop="title" required>
        <RichEditor
          :id="-1"
          :cont="question.title"
          @updateContent="updateContent"
        ></RichEditor>
      </el-form-item>
      <el-form-item label="选项：" required>
        <el-form-item
          :label="idx + `、`"
          :key="idx"
          v-for="(item, idx) in question.items"
          class="item question-item-label"
          label-width="50px"
        >
          <el-input v-model="item.prefix" style="width: 50px"></el-input>
          <RichEditor
            :id="idx"
            :cont="item.content"
            @updateContent="updateContent"
          ></RichEditor>
          <el-button
            type="danger"
            size="mini"
            class="question-item-remove"
            icon="el-icon-delete"
            @click="questionItemRemove(idx)"
          ></el-button>
        </el-form-item>
      </el-form-item>
      <el-form-item label="答案：" prop="answer" required>
        <draggable v-model="question.answer">
          <transition-group>
            <div
              class="answerBox"
              v-for="element in question.answer"
              :key="element.prefix"
            >
              <div class="ql-container ql-snow">
                <span>{{ element.prefix }}</span>
                <div class="content ql-editor" v-html="element.content" />
              </div>
            </div>
          </transition-group>
        </draggable>
      </el-form-item>
      <el-form-item label="解析：" prop="analyze" required>
        <RichEditor
          :id="-2"
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
      <el-form-item label="难度：" required>
        <el-rate
          v-model="question.difficult"
          class="question-item-rate"
        ></el-rate>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitQuestion">提交</el-button>
        <el-button @click="resetQuestion">重置</el-button>
        <el-button type="success" @click="questionItemAdd">添加选项</el-button>
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
import "quill/dist/quill.core.css";
import "quill/dist/quill.snow.css";
import "quill/dist/quill.bubble.css";
import RichEditor from "@/components/RichEditor/index.vue";
import draggable from "vuedraggable";
import QuestionShow from "@/components/QuestionShow/index.vue";
export default {
  name: "sort",
  components: {
    RichEditor,
    draggable,
    QuestionShow,
  },
  created() {
    let myRow = this.$route.params.question;
    if (myRow != null) {
      this.isEdit = true;
      let myItems = myRow.items;
      let Items = myItems.split("<sep1>");
      this.question.items = [];
      this.question.id = myRow.qid;
      for (let i = 0; i < Items.length; i++) {
        let item = Items[i].split("<sep2>");
        let obj = { prefix: item[0], content: item[1] };
        this.question.items.push(obj);
      }
      var answer = [];
      for (let i = 0; i < myRow.answer.length; i++) {
        var obj = {};
        obj.prefix = myRow.answer[i];
        for (let j = 0; j < this.question.items.length; j++) {
          if (this.question.items[j].prefix === obj.prefix) {
            obj.content = this.question.items[j].content;
          }
        }
        answer.push(obj);
      }
      this.question.title = myRow.title;
      this.question.answer = answer;
      this.question.analyze = myRow.analysis;
      this.question.score = myRow.score;
      this.question.difficult = myRow.difficult;
      this.question.questionType = myRow.qtype;
    }
  },
  data() {
    return {
      isEdit: false,
      question: {
        id: null,
        questionType: 5,
        difficult: 0,
        title: "",
        items: [
          { prefix: "A", content: "" },
          { prefix: "B", content: "" },
          { prefix: "C", content: "" },
          { prefix: "D", content: "" },
        ],
        answer: [
          { prefix: "A", content: "" },
          { prefix: "B", content: "" },
          { prefix: "C", content: "" },
          { prefix: "D", content: "" },
        ],

        analyze: "",
        score: "",
      },
      rules: {
        title: [{ required: true, message: "请输入题干", trigger: "blur" }],
        analyze: [{ required: true, message: "请输入解析", trigger: "blur" }],
        score: [{ required: true, message: "请输入分数", trigger: "blur" }],
        answer: [{ required: true, message: "请输入答案", trigger: "blur" }],
      },
      questionVisible: false,
    };
  },
  methods: {
    questionItemRemove(idx) {
      this.question.items.splice(idx, 1);
      this.question.answer = this.question.items;
    },
    questionItemAdd() {
      let items = this.question.items;
      let newLastPrefix = null;
      if (items.length > 0) {
        let last = items[items.length - 1];
        newLastPrefix = String.fromCharCode(last.prefix.charCodeAt() + 1);
      } else {
        newLastPrefix = "A";
      }
      items.push({ prefix: newLastPrefix, content: "" });
      this.question.answer = this.question.items;
    },
    updateContent({ id, content }) {
      if (id === -1) {
        this.question.title = content;
      } else if (id === -2) {
        this.question.analyze = content;
      } else if (id >= 0) {
        this.question.items[id].content = content;
      }
      this.question.answer = this.question.items;
    },
    submitQuestion() {
      let that = this;
      this.$refs.question.validate((valid) => {
        if (valid) {
          var items = "";
          for (let i = 0; i < this.question.items.length - 1; i++) {
            items +=
              this.question.items[i].prefix +
              "<sep2>" +
              this.question.items[i].content +
              "<sep1>";
          }
          items +=
            this.question.items[this.question.items.length - 1].prefix +
            "<sep2>" +
            this.question.items[this.question.items.length - 1].content;
          var answer = "";
          for (let i = 0; i < this.question.answer.length; i++) {
            answer += this.question.answer[i].prefix;
          }
          if (!this.isEdit) {
            this.apis.question
              .submitQuestion(
                sessionStorage.getItem("teacherUsername"),
                sessionStorage.getItem("teacherUsername"),
                this.question.title,
                answer,
                this.question.analyze,
                items,
                this.question.score,
                this.question.difficult,
                5
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
            this.apis.question
              .editQuestion(
                this.question.id,
                this.question.title,
                answer,
                this.question.analyze,
                items,
                this.question.score,
                this.question.difficult,
                5
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
          }
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
        questionType: 1,
        difficult: 0,

        title: "",
        items: [
          { prefix: "A", content: "" },
          { prefix: "B", content: "" },
          { prefix: "C", content: "" },
          { prefix: "D", content: "" },
        ],
        answer: [
          { prefix: "A", content: "" },
          { prefix: "B", content: "" },
          { prefix: "C", content: "" },
          { prefix: "D", content: "" },
        ],

        analyze: "",
        score: "",
      };
    },
  },
};
</script>

<style scoped>
#sort {
  width: 100%;
  min-height: 50vh;
  box-sizing: border-box;
  padding: 20px;
}
.answerBox {
  cursor: pointer;
  min-height: 40px;
  margin-bottom: 15px;
  background: #ffff;
  border: 1px solid #dcdfe6;
  padding: 5px 10px;
  border-radius: 4px;
}
.answerBox:hover {
  background: #f5f7fa;
}
.ql-container.ql-snow {
  border: none !important;
}
</style>