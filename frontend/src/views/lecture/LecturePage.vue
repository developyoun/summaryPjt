<template>
  <v-row class="lecture-container">
    <v-col class="video-container">
      <div class="lecture-video">
        <lecture-video @player="getPlayer" />
        <!-- <youtube
          v-if="lecture"
          fitParent
          :video-id="lecture.video_id"
          ref="youtube"
        ></youtube> -->
      </div>
      <!-- <iframe
        class="lecture-video"
        width="100%"
        height="371"
        :src="videoURL"
        frameborder="0"
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
        allowfullscreen
      ></iframe> -->
    </v-col>
    <v-col class="editor-container">
      <v-row style="margin: 0px">
        <v-col class="menu-col">
          <v-btn class="menu-btn" tile @click="toNoteTab">노트</v-btn>
        </v-col>
        <v-col class="menu-col">
          <v-btn class="menu-btn" tile @click="toQuizTab">문제</v-btn>
        </v-col>
        <v-col class="menu-col">
          <v-btn class="menu-btn" tile @click="toListTab">목록</v-btn>
        </v-col>
      </v-row>
      <text-editor
        :player="player"
        v-if="!$route.query.tab || $route.query.tab === 'note'"
      />
      <quiz v-else-if="$route.query.tab === 'quiz'" />
      <curriculum v-else-if="$route.query.tab === 'list'" />
    </v-col>
  </v-row>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import TextEditor from "@/components/lecture/TextEditor.vue";
import Quiz from "@/components/lecture/Quiz.vue";
import Curriculum from "@/components/lecture/Curriculum.vue";
import LectureVideo from "@/components/lecture/LectureVideo.vue";
import { namespace } from "vuex-class";
import { Lecture } from "../../store/Lectures.interface";

const LecturesModule = namespace("LecturesModule");

@Component({
  components: {
    TextEditor,
    Curriculum,
    Quiz,
    LectureVideo
  }
})
export default class LecturePage extends Vue {
  @LecturesModule.State lecture!: Lecture;
  @LecturesModule.Action FETCH_ALL_CAPTURE_IMAGES: any;
  @LecturesModule.Action FETCH_LECTURE_DETAIL: any;
  @LecturesModule.Action FETCH_PROBLEM_LIST: any;

  player: { [key: string]: any } = {};

  playerVars = {
    start: 15
  };

  toNoteTab() {
    this.$router.replace({ name: "LecturePage", query: { tab: "note" } });
  }
  toQuizTab() {
    this.$router.replace({ name: "LecturePage", query: { tab: "quiz" } });
  }
  toListTab() {
    this.$router.replace({ name: "LecturePage", query: { tab: "list" } });
  }
  getPlayer(v: { [key: string]: any }) {
    this.player = v;
  }
  scroll(event: Event) {
    event.preventDefault();
    event.stopPropagation();
  }
  async preventScroll() {
    const video = await document.querySelector(".video-container");
    video?.addEventListener("scroll", this.scroll);
    video?.addEventListener("mousewheel", this.scroll);
  }

  @Watch("lecture", { immediate: true, deep: true })
  fetchAllCapture() {
    if (this.lecture) {
      this.FETCH_PROBLEM_LIST(this.lecture.video_id);
      // this.FETCH_ALL_CAPTURE_IMAGES(this.videoURL);
      this.FETCH_ALL_CAPTURE_IMAGES(this.lecture.video_id);
    }
  }

  mounted() {
    this.preventScroll();
  }
  @Watch("$route", { immediate: true })
  fetchLectureDetail() {
    this.FETCH_LECTURE_DETAIL(this.$route.params.playId);
  }
}
</script>

<style scoped>
.lecture-container {
  height: 100%;
}
.video-container {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 50%;
  min-width: 650px;
  height: 100%;
  padding: 0;
  background-color: black;
  overflow: hidden;
  overflow-x: hidden;
  overflow-y: hidden;
}
.editor-container {
  width: 50%;
  min-width: 300px;
  height: 100%;
  padding: 0;
}
.lecture-video {
  position: fixed;
  top: 30%;
}
.menu-col {
  padding: 0;
}
.menu-btn {
  width: 100%;
}
</style>
