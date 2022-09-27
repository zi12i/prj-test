<template>
  <v-dialog persistent max-width="1000" v-model="dialog">
    <v-card class="modal">
      <div class="modalHeader">
        <h2>카테고리 추가</h2>
        <v-btn icon text @click="closeModal">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </div>
      <v-divider></v-divider>

      <v-card-text>
        <ValidationObserver v-slot="{ handleSubmit, invalid, validate }">
          <form class="formStyle" @submit.prevent="handleSubmit(addCategory)">
            <ValidationProvider
              name="카테고리명"
              rules="required|min:3|max:100"
              v-slot="{ errors }"
            >
              <v-text-field
                dense
                filled
                label="카테고리명(필수 항목)"
                :error-messages="errors"
                v-model="formData.title"
              ></v-text-field>
            </ValidationProvider>

            <ValidationProvider
              name="설명"
              rules="required|min:3|max:5000"
              v-slot="{ errors }"
            >
              <v-text-field
                dense
                filled
                label="설명"
                :error-messages="errors"
                v-model="formData.description"
              ></v-text-field>
            </ValidationProvider>

            <v-btn
              depressed
              block
              color="blue"
              class="white--text"
              :disabled="invalid || !validate"
              type="submit"
              >저장</v-btn
            >
          </form>
        </ValidationObserver>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>
<script>
import axios from "axios";
import Vaildate from "@/mixins/Validate.vue";

export default {
  name: "addModal",
  mixins: [Vaildate],
  props: {
    openDialog: Boolean,
  },
  data: () => ({
    categories: [], // 카테고리 데이터
    categoryTitles: [], // 카테고리 타이틀만

    formData: {
      id: "",
      title: "",
      description: "",
    },

    categoryLoading: false,
    uploading: false,
    uploaded: false,
    saveLoading: false,
  }),
  computed: {
    dialog(props) {
      console.log(props.openDialog);
      return props.openDialog;
    },
  },
  methods: {
    closeModal() {
      this.$emit("closeDialog");
    },

    async addCategory() {
      if (this.loading) return;
      this.loading = true;

      const axiosBody = {
        title: this.formData.title,
        description: this.formData.description,
      };
      console.log("/categories - axiosBody : ", axiosBody);

      await axios
        .post(process.env.VUE_APP_API + "/categories", axiosBody, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then(async (response) => {
          console.log("categories - response : ", response);
          this.$router.go();
        })
        .catch((error) => {
          console.log("categories - error : ", error);
        })
        .finally(() => {
          this.loading = false;
        });
    },

    async saveCategory() {
      this.saveLoading = true;
      const categoryId = this.categories.find((category) => {
        if (category.title === this.formData.category) return true;
      })._id;

      const axiosBody = {
        categoryId: categoryId,
        title: this.formData.title,
        description: this.formData.description,
      };
      console.log("editCategory - axiosBody : ", axiosBody);

      const categoryFormData = new FormData();
      categoryFormData.append("title", this.formData.title);
      categoryFormData.append("description", this.formData.description);
      categoryFormData.append("categoryId", categoryId);
      for (let item of categoryFormData.entries()) {
        console.log(item);
      }

      await axios
        .put(this.url, categoryFormData, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          console.log("saveVideo - response : ", response);
        })
        .catch((error) => {
          console.log("saveVideo - error : ", error);
        })
        .finally(() => {
          this.closeModal();
        });
    },
  },
};
</script>
<style>
.modal {
  box-sizing: border-box;
  padding: 20px 30px;
}
.modalHeader {
  align-items: center;
  display: flex;
  justify-content: space-between;
  padding-bottom: 10px;
}
.fileInputStyle {
  margin: 20px auto 0px auto;
  width: 80%;
}
.formStyle {
  margin: 20px auto 0px auto;
}
</style>
