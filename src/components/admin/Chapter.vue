<template>
  <div>
    <div>
      <div class="flex" style="align-items: center;">
        <h1 class="flex1" style="text-transform: uppercase;">Chapter</h1>
        <div>
          <b-button class="float-right" variant="primary" @click.prevent="onOpenModal('chapter', true)">+</b-button>
        </div>
      </div>
      <div class="chapter_items" v-for="(chapter, index) in chapters" :key="index">
        <div class="flex chapter c_pointer">
          <div class="chapter_title flex1" @click="selectedChapter(index)"><span>Chapter {{ index + 1 }}: {{ chapter.name }}</span></div>
          <div class="flex">
            <b-button class="mr-1 btn_action" variant="primary" @click.prevent="onOpenModal('session', true, chapter)">+</b-button>
            <button class="btn btn-info mr-1 btn_action" @click.prevent="onOpenModal('chapter', false, chapter)"><b-icon icon="pencil-fill" variant="white" /></button>
            <button class="btn btn-danger btn_action" @click.prevent="onDeleteData('chapter', chapter)"><b-icon icon="trash-fill" variant="white" /></button>
          </div>
        </div>
        <div v-if="show.includes(index)">
          <div>
            <li v-for="(session, indexS) in chapter.sessions" v-show="session.id" :key="indexS" class="flex" style="align-items: center; height: 80px;">
              <div>
                <video :src="session.url_video" height="50" width="100" controls />
              </div>
              <div class="flex1 pl-3">
                <span style="font-size: 15px;">{{ session.name }}</span>
              </div>
              <div>
                <div style="width: 50px;">
                  <button
                    class="btn btn-info mb-1 btn_action"
                    @click="onOpenModal('session', false, session)"
                  ><b-icon icon="pencil-fill" variant="white" /></button>
                  <button class="btn btn-danger btn_action" @click.prevent="onDeleteData('session', session)"><b-icon icon="trash-fill" variant="white" /></button>
                </div>
              </div>
            </li>
          </div>
        </div>
      </div>
    </div>
    <div>
      <b-modal
        id="modal-create-chapter"
        ref="modal"
        v-model="isModalFormOpen"
        :title="titleForm"
        hide-footer
        hide-header-close
      >
        <modal-form-chapter
          v-if="modal==='chapter'"
          :is-add="modalOpts.isAdd"
          :data="modalOpts.data"
          @onadd="onAddData($event)"
          @onupdate="onEditData($event)"
          @cancelForm="onCancelForm"
        />
        <modal-form-session
          v-else
          :is-add="modalOpts.isAdd"
          :data="modalOpts.data"
          @onadd="onAddSession($event)"
          @onupdate="onEditSession($event)"
          @cancelForm="onCancelForm"
        />
      </b-modal>
      <b-modal
        v-if="isModelDeleteOpen"
        id="modal-delete-category"
        v-model="isModelDeleteOpen"
        title="Please Confirm"
        size="sm"
        button-size="sm"
        ok-variant="danger"
        ok-title="YES"
        cancel-title="NO"
        footer-class="p2"
        hide-header-close
        @cancel="onCancelDelete()"
        @ok="onConfirmDeleteData()"
      >Are you sure you want to remove {{ modal }}: {{ selectedDelete.name }}?</b-modal>
    </div>
  </div>
</template>
<script>
import { getChapterList } from '../../api/course'
import { addChapter, updateChapter, deleteChapter } from '../../api/chapter'
import { addSession, updateSession, deleteSession } from '../../api/session'
import ModalFormChapter from '../admin/modal/ModalFormChapter.vue'
import ModalFormSession from '../admin/modal/ModalFormSession.vue'
export default {
  components: {
    ModalFormChapter,
    ModalFormSession
  },
  props: {
    courseId: {
      type: Number,
      default: null
    }
  },
  data() {
    return {
      chapters: [],
      selectedDelete: null,
      modalOpts: {},
      isModalFormOpen: null,
      isModelDeleteOpen: null,
      titleForm: '',
      modal: null,
      show: []
    }
  },
  async created() {
    this.chapters = await getChapterList(this.courseId)
  },
  methods: {
    onOpenModal(modal, isAdd, data) {
      this.modal = modal
      this.modalOpts.isAdd = isAdd
      this.handleData(modal, isAdd, data)
      this.isModalFormOpen = true
    },
    handleData(modal, isAdd, data) {
      if (modal === 'chapter') {
        this.titleForm = isAdd ? 'Create chapter' : 'Update chaper'
        this.modalOpts.data = {
          'id': isAdd ? 0 : data.id,
          'name': isAdd ? null : data.name,
          'created_by': this.$store.state.Admin.myInfo.user_name,
          'updated_by': this.$store.state.Admin.myInfo.user_name
        }
      } else {
        this.titleForm = isAdd ? 'Create session' : 'Update session'
        this.modalOpts.data = {
          'id': isAdd ? 0 : data.id,
          'name': isAdd ? null : data.name,
          'url_video': isAdd ? null : data.url_video,
          'about': isAdd ? null : data.about,
          'time': isAdd ? null : data.time,
          'chapter_id': isAdd ? data.id : data.chapter_id,
          'created_by': this.$store.state.Admin.myInfo.user_name,
          'updated_by': this.$store.state.Admin.myInfo.user_name
        }
      }
    },
    onAddData: async function(data) {
      this.$store.commit('SET_LOADING')
      try {
        const body = {
          'name': data.name.trim(),
          'course_id': this.courseId,
          'created_by': this.$store.state.Admin.myInfo.user_name,
          'updated_by': this.$store.state.Admin.myInfo.user_name
        }
        await addChapter(body)
        this.res('success', 'Chapter successfully created.')
        this.chapters = await getChapterList(this.courseId)
      } catch (error) {
        this.res('danger', error.response?.data?.message || error.message)
      }
      this.isModalFormOpen = false
      this.$store.commit('SET_DONE_LOADING')
    },
    async onEditData(data) {
      this.$store.commit('SET_LOADING')
      try {
        await updateChapter(data)
        this.res('success', 'Chapter successfully updated.')
        this.chapters = await getChapterList(this.courseId)
      } catch (error) {
        this.res('danger', error.response?.data?.message || error.message)
      }
      this.isModalFormOpen = false
      this.$store.commit('SET_DONE_LOADING')
    },
    onAddSession: async function(data) {
      try {
        const body = {
          'name': data.name.trim(),
          'url_video': data.url_video,
          'chapter_id': data.chapter_id,
          'time': data.time,
          'about': data.about,
          'created_by': this.$store.state.Admin.myInfo.user_name,
          'updated_by': this.$store.state.Admin.myInfo.user_name
        }
        await addSession(body)
        this.res('success', 'Session successfully created.')
        this.chapters = await getChapterList(this.courseId)
      } catch (error) {
        this.res('danger', error.response?.data?.message || error.message)
      }
      this.isModalFormOpen = false
      this.$store.commit('SET_DONE_LOADING')
    },
    onEditSession: async function(data) {
      this.$store.commit('SET_LOADING')
      try {
        await updateSession(data)
        this.res('success', 'Session successfully updated.')
        this.chapters = await getChapterList(this.courseId)
      } catch (error) {
        this.res('danger', error.response?.data?.message || error.message)
      }
      this.isModalFormOpen = false
      this.$store.commit('SET_DONE_LOADING')
    },
    onCancelForm() {
      this.isModalFormOpen = false
    },
    onDeleteData(modal, data) {
      this.modal = modal
      this.selectedDelete = data
      this.isModelDeleteOpen = true
    },
    onCancelDelete() {
      this.isModelDeleteOpen = false
      this.selectedDelete = null
    },
    onConfirmDeleteData: async function() {
      this.$store.commit('SET_LOADING')
      try {
        if (this.modal === 'chapter') {
          await deleteChapter(this.selectedDelete.id)
          this.res('success', 'Chapter successfully deleted.')
        } else {
          await deleteSession(this.selectedDelete.id)
          this.res('success', 'Session successfully deleted.')
        }
      } catch (error) {
        this.res('danger', error.response?.data?.message || error.message)
      } finally {
        this.isModelDeleteOpen = false
        this.chapters = await getChapterList(this.courseId)
      }
      this.$store.commit('SET_DONE_LOADING')
    },
    selectedChapter(index) {
      if (this.show.includes(index)) {
        this.show.splice(this.show.indexOf(index), 1)
      } else {
        this.show.push(index)
      }
    },
    res(status, message) {
      const res = {
        status: status,
        message: message
      }
      this.$emit('resaction', res)
    }
  }
}
</script>

<style lang="scss" scoped>
.chapter_items {
  .chapter:hover{
    background-color: #64D9D6;
  }
}
.chapter {
  align-items: center;
  height: 50px;
  border-bottom: 1px solid;
}
.chapter_title {
  font-size: 13px;
  text-transform: uppercase;
  font-weight: bold;
}
.btn_action {
  font-size: 65%;
}
</style>
