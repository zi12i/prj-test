<template>
  <div>
    <v-btn @click="openCategoryModal"><v-icon small class="pr-1">mdi-plus-circle-outline</v-icon>카테고리 추가</v-btn>
    <h2>채널 콘텐츠</h2>
        <v-data-table :headers="headers" :items="categories">
          <template #item="{ item }">
            <tr>
              <td>{{ item.title }}</td>
              <td>{{ item.description }}</td>
              <td>
                <span class="mr-3">
                  <v-btn @click="openCategoryEditModal(item)"><v-icon small class="pr-1">mdi-pencil</v-icon>카테고리수정</v-btn>
    
                </span>
              </td>
            </tr>
          </template>
        </v-data-table>
    <CategoryModal
      :openDialog="statusModal"
      v-on:closeDialog="closeCategoryModal"></CategoryModal>
    <CategoryEditModal
      :openDialog="statusEditModal"
      v-on:closeDialog="closeCategoryEditModal"
      :toEidtID="toEdit"></CategoryEditModal>
  </div>
</template>

<script>
import axios from 'axios'
import SetFormat from '@/mixins/SetFormat.vue'
import CategoryModal from '@/components/Modal/CategoryModal.vue';
import CategoryEditModal from '@/components/Modal/CategoryEditModal.vue';
export default {
  name: 'Category',
  mixins: [SetFormat],
 components: {
    CategoryModal,
    CategoryEditModal,
  },
  data: () => ({
    tab: null,
    statusModal: false,
    statusEditModal: false,
    toEdit: '',
    headers: [
      {
        text: 'Categories',
        align: 'start',
        value: 'title'
      },
      { text: '설명', value: 'title' },
      { text: '수정', value: 'editBtn' },
    ],
    categories: [],
    components: {
    },
  }),
  mounted() {
    this.getCategories()
  },
  methods: {
    openCategoryModal() {
      this.statusModal = true;
      console.log('-- open : ', this.statusModal);
    },
    closeCategoryModal() {
      this.statusModal = false;
      console.log('-- close : ', this.statusModal);
    },
    openCategoryEditModal(item) {
      this.statusEditModal = true;
      this.toEdit = item._id;
      console.log('-- open : ', this.statusEditModal);
      console.log('-- open item : ', item);
      console.log('-- toEdit : ', this.toEdit)
    },
    closeCategoryEditModal() {
      this.statusEditModal = false;
      console.log('-- close : ', this.statusEditModal);
    },
    async getCategories() {
      // 비디오모달에서 가쟈온 코드
      this.categoryLoading = true;
      await axios
        .get(process.env.VUE_APP_API + '/categories', {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
        })
        .then((response) => {
          console.log('getCategories - response : ', response.data.data);
          this.categories = response.data.data;
        })
        .catch((error) => {
          console.log('getCategories - error : ', error);
        })
        .finally(() => {
          this.categoryLoading = false;
        });
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
