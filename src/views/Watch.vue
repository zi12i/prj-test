<template>
  <v-container
    id="Watch"
    class="fill-height"
    fluid
  >
    <div class="watchSection fill-height">
      <div class="watchContainer">
        <v-skeleton-loader
          large
          tile
          :loading="videoLoading"
        >
          <v-responsive>
            <video
              controls
              style="height: 100%; width: 100%"
            >
              <source
                :src="videoUrl !== null && videoUrl"
                type="video/mp4"
              />
            </video>
          </v-responsive>
        </v-skeleton-loader>

        <div class="videoInfo">
          <h2 class="videoTitle">{{ watchVideo.title }}</h2>
        </div>
      </div>
    </div>

    <div class="listSection fill-height">
      <div class="listContainer">
        <VideoListCard
          v-for="video in videos"
          :key="video.id"
          :video="video"
          :channel="video.userId"
        ></VideoListCard>
      </div>
    </div>
    <div>
      <!-- 댓글 뽑는리스트부분 -->
      <div id="app">
        <v-app id="inspire">
          <div>
            <ValidationObserver v-slot="{ handleSubmit, invalid, validate }">
              <form @submit.prevent="handleSubmit(addComment)">
                <ValidationProvider
                  name="댓글"
                  rules="required|min:1|max:5000"
                  v-slot="{ errors }"
                >
                  <v-text-field
                    v-model="formData.text"
                    placeholder="댓글 추가.."
                    :error-messages="errors"
                    required
                  ></v-text-field>
                </ValidationProvider>
                <v-btn
                  depressed
                  block
                  color="blue"
                  class="white--text"
                  :disabled="invalid || !validate"
                  type="submit"
                >
                  댓글
                </v-btn>
              </form>
            </ValidationObserver>
          </div>
          <v-list three-line>
            <template v-for="(item, index) in items">
              <v-subheader
                v-if="item.header"
                :key="item.header"
                v-text="item.header"
              ></v-subheader>

              <v-divider
                v-else-if="item.divider"
                :key="index"
                :inset="item.inset"
              ></v-divider>

              <v-list-item
                v-else
                :key="item.title"
              >
                <v-list-item-avatar
                  small
                  color="red white--text"
                  class="align-center d-flex font-weight-bold justify-center"
                >
                  <span>{{ item.avatar }}</span>
                </v-list-item-avatar>

                <v-list-item-content>
                  <v-list-item-title v-html="item.title"></v-list-item-title>
                  <v-list-item-subtitle
                    v-html="item.subtitle"
                    style="margin: 10px 0 20px 0"
                  ></v-list-item-subtitle>

                  <div
                    v-if="
                      editTextbox === true && item.data.id === clickedComment.id
                    "
                  >
                    <ValidationObserver
                      v-slot="{ handleSubmit, invalid, validate }"
                    >
                      <form
                        @submit.prevent="
                          handleSubmit(() => {
                            editComment(item);
                          })
                        "
                      >
                        <ValidationProvider
                          name="댓글"
                          rules="required|min:1|max:5000"
                          v-slot="{ errors }"
                        >
                          <v-text-field
                            v-model="formData.text"
                            placeholder="수정하실 내용을 입력하세요."
                            :error-messages="errors"
                            required
                          ></v-text-field>
                        </ValidationProvider>
                        <v-btn
                          depressed
                          block
                          color="blue"
                          class="white--text"
                          :disabled="invalid || !validate"
                          type="submit"
                        >
                          수정
                        </v-btn>
                      </form>
                    </ValidationObserver>
                  </div>
                  <div class="btnOrganiser">
                    <v-btn
                      elevation="2"
                      small
                      >답글</v-btn
                    >
                    <v-btn
                      elevation="2"
                      small
                      @click="openReplySection(item)"
                      >답글 보기</v-btn
                    >
                    <v-spacer></v-spacer>
                    <v-btn
                      elevation="2"
                      small
                      @click="openEditSection(item)"
                      ><v-icon>mdi-pencil-outline</v-icon>수정</v-btn
                    ><v-btn
                      elevation="2"
                      small
                      @click="deleteComment(item)"
                      ><v-icon>mdi-delete-outline</v-icon>삭제</v-btn
                    >
                  </div>
                  <!-- 답글보기 누르면 이게 열리게 할거임 -->
                  <div
                    v-if="
                      replyBox === true && item.data.id === selectedComment.id
                    "
                  >
                    <template v-for="(reply, index) in item.replies">
                      <v-subheader
                        v-if="reply.header"
                        :key="reply.header"
                        v-text="reply.header"
                      ></v-subheader>

                      <v-divider
                        v-else-if="reply.divider"
                        :key="index"
                        :inset="reply.inset"
                      ></v-divider>

                      <v-list-item
                        v-else
                        :key="reply.title"
                      >
                        <v-list-item-avatar
                          small
                          color="red white--text"
                          class="align-center d-flex font-weight-bold justify-center"
                        >
                          <span>{{ reply.userId.channelName.charAt(0) }}</span>
                        </v-list-item-avatar>

                        <v-list-item-content>
                          <v-list-item-title
                            v-html="reply.userId.channelName"
                          ></v-list-item-title>
                          <v-list-item-subtitle
                            v-html="reply.text"
                            style="margin: 10px 0 20px 0"
                          ></v-list-item-subtitle>
                          <div class="btnOrganiser">
                            <v-btn
                              elevation="2"
                              x-small
                              @click="openReplyEditSection(item)"
                              ><v-icon>mdi-pencil-outline</v-icon>수정</v-btn
                            ><v-btn
                              elevation="2"
                              x-small
                              @click="deleteReplyComment(item)"
                              ><v-icon>mdi-delete-outline</v-icon>삭제</v-btn
                            >
                          </div>
                          <!-- 답글수정 누르면 열릴거 -->
                          <div
                            v-if="
                              editTextbox === true &&
                              item.data.id === clickedReplyComment.id
                            "
                          >
                            <ValidationObserver
                              v-slot="{ handleSubmit, invalid, validate }"
                            >
                              <form
                                @submit.prevent="
                                  handleSubmit(() => {
                                    editComment(item);
                                  })
                                "
                              >
                                <ValidationProvider
                                  name="댓글"
                                  rules="required|min:1|max:5000"
                                  v-slot="{ errors }"
                                >
                                  <v-text-field
                                    v-model="formData.text"
                                    placeholder="수정하실 내용을 입력하세요."
                                    :error-messages="errors"
                                    required
                                  ></v-text-field>
                                </ValidationProvider>
                                <v-btn
                                  depressed
                                  block
                                  color="blue"
                                  class="white--text"
                                  :disabled="invalid || !validate"
                                  type="submit"
                                >
                                  수정
                                </v-btn>
                              </form>
                            </ValidationObserver>
                          </div>
                        </v-list-item-content>
                      </v-list-item>
                    </template>
                  </div>
                </v-list-item-content>
              </v-list-item>
            </template>
          </v-list>
        </v-app>
      </div>
    </div>
  </v-container>
</template>
<script>
import axios from "axios";
import Vaildate from "@/mixins/Validate.vue";
import VideoListCard from "@/components/VideoListCard.vue";

export default {
  name: "Watch",
  mixins: [Vaildate],
  data: () => ({
    comment: "",
    videos: [],
    watchVideo: {},
    watchComment: {},
    watchReply: {},
    videoUrl: null,
    videoLoading: false,
    editTextbox: false,
    replyBox: false,
    items: [],
    formData: {
      text: "",
      videoId: "",
      userId: "",
    },

    clickedComment: {},
    clickedReplyComment: {},
    selectedComment: {},
  }),
  components: {
    VideoListCard,
  },
  watch: {
    $route(to, from) {
      if (to.path != from.path) {
        this.getWatchData(this.$route.params.id);
        this.getComments(this.$route.params.id);
      }
    },
    watchVideo() {
      this.videoUrl = `${process.env.VUE_APP_URL}/uploads/videos/${this.watchVideo.url}`;
    },
  },
  methods: {
    async getWatchData(id) {
      console.log(this.$route.params.id);
      this.videoLoading = true;
      this.watchVideo = {};

      await axios
        .get(process.env.VUE_APP_API + `/videos/${id}`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          console.log(
            "getWatchData - response : ",
            response,
            response.data.data
          );
          this.watchVideo = response.data.data;
          this.videoLoading = false;
        })
        .catch((error) => {
          console.log("getWatchData - error : ", error);
        });
    },
    //댓글 조회 기능
    async getComments(id) {
      this.watchComment = {};
      console.log("아이디: ", id);
      await axios
        .get(process.env.VUE_APP_API + `/comments/${id}/videos`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          console.log(
            "getCommentData - response : ",
            response,
            response.data.data
          );
          this.watchComment = response.data.data;
          console.log("이러면 댓글 뽑히나?", response.data.data[0]);

          let count = 0;
          const newArray = [{ header: " 댓글" }];
          response.data.data.forEach((thisItem) => {
            newArray.push({ divider: true, inset: true });

            const pushData = {
              avatar: thisItem.userId.channelName.charAt(0),
              title: thisItem.userId.channelName,
              subtitle: thisItem.text,
              data: thisItem,
              replies: thisItem.replies,
            };

            newArray.push(pushData);

            count++;
            if (count === response.data.data.length) {
              console.log("newArray : ", newArray);
              this.items = newArray;
            }
          });
        })
        .catch((error) => {
          console.log("getCommentData - error : ", error);
        });
    },
    async getVideos() {
      await axios
        .get(process.env.VUE_APP_API + "/videos/public", {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          console.log("getVideos - response : ", response, response.data.data);
          this.videos = response.data.data;
        })
        .catch((error) => {
          console.log("getVideos - error : ", error);
        });
    },
    // 댓글 등록
    async addComment() {
      if (this.loading) return;
      this.loading = true;

      const axiosBody = {
        text: this.formData.text,
        videoId: this.$route.params.id,
      };
      console.log("/comment - axiosBody : ", axiosBody);

      await axios
        .post(process.env.VUE_APP_API + "/comments", axiosBody, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then(async (response) => {
          console.log("comment - response : ", response);
          this.getComments(this.$route.params.id);
        })
        .catch((error) => {
          console.log("comment - error : ", error);
        })
        .finally(() => {
          this.loading = false;
        });
    },

    // 댓글 수정
    async editComment(item) {
      this.saveLoading = true;
      this.categoryToEdit = this.postCategory;

      console.log("itemdata : ", item.data.id);

      const axiosBody = {
        text: this.formData.text,
        commentId: item.data.id,
      };
      console.log("editComment - axiosBody : ", axiosBody);

      await axios
        .put(
          `${process.env.VUE_APP_API}/comments/${axiosBody.commentId}`,
          axiosBody,
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          }
        )
        .then((response) => {
          console.log("editComment - response : ", response);
          console.log("악쉬오쑤 바뒤: ", axiosBody);
        })
        .catch((error) => {
          console.log("editComment - error : ", error);
        })
        .finally(() => {
          this.getComments(this.$route.params.id);
          this.editTextbox = false;
        });
    },
    // 대댓글 수정기능
    async editReplyComment(item) {
      this.saveLoading = true;
      this.categoryToEdit = this.postCategory;

      console.log("itemdata : ", item.data.id);

      const axiosBody = {
        text: this.formData.text,
        commentId: item.data.id,
      };
      console.log("editComment - axiosBody : ", axiosBody);

      await axios
        .put(
          `${process.env.VUE_APP_API}/comments/${axiosBody.commentId}`,
          axiosBody,
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          }
        )
        .then((response) => {
          console.log("editComment - response : ", response);
          console.log("악쉬오쑤 바뒤: ", axiosBody);
        })
        .catch((error) => {
          console.log("editComment - error : ", error);
        })
        .finally(() => {
          this.getComments(this.$route.params.id);
          this.editTextbox = false;
        });
    },

    //삭제기능
    // 한개 남았을때는 새로고침해야 삭제됨

    async deleteComment(item) {
      this.saveLoading = true;
      this.categoryToEdit = this.postCategory;

      console.log("itemdata : ", item.data.id);

      const axiosBody = {
        text: this.formData.text,
        commentId: item.data.id,
      };
      console.log("deleteComment - axiosBody : ", axiosBody);
      console.log("이게 댓글 아이디 아니야?", item);

      await axios
        .delete(`${process.env.VUE_APP_API}/comments/${axiosBody.commentId}`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          console.log("deleteComment - response : ", response);
        })
        .catch((error) => {
          console.log("deleteComment - error : ", error);
        })
        .finally(() => {
          this.getComments(this.$route.params.id);
        });
    },
    openEditSection(item) {
      this.clickedComment = item.data;
      this.editTextbox = true;
    },
    openReplyEditSection(item) {
      this.clickedReplyComment = item.data;
      this.editReplyTextbox = true;
    },
    openReplySection(item) {
      this.selectedComment = item.data;
      this.replyBox = true;
    },
  },
  mounted() {
    this.getVideos();
    this.getWatchData(this.$route.params.id);
    this.getComments(this.$route.params.id);
  },
};
</script>
<style>
#Watch {
  display: grid;
  grid-template-columns: 70% 30%;
}

.videoInfo {
  box-sizing: border-box;
  padding: 10px;
  width: 100%;
}

.listSection {
  align-items: center;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  padding-left: 10px;
  width: 100%;
}
.listContainer {
  align-items: center;
  box-sizing: border-box;
  row-gap: 5px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  padding-left: 10px;
  width: 100%;
}
.inputStyle {
  width: 300px;
}
.btnOrganiser {
  display: flex;
  justify-content: flex-end;
}
.btnLeft {
  size: 10px;
}
</style>
