<template>
  <div class="te_chapter">
    <div class="te_form_chapter" :class="{ te_active_form_chapter: isShowForm }">
      <component :is="comOpts.comName" :is-add="comOpts.isAdd" :data="comOpts.data" @onadd="onAddData($event)" @onupdate="onEditData($event)" @oncloseform="closeForm($event)" />
    </div>
    <div class="te_chapter_heading">
      <div class="w50">
        <h1>Chapter</h1>
      </div>
      <div class="w50 txt_right">
        <b-button variant="primary" @click="onOpenForm('FormChapter', true, null)">+ Chapter</b-button>
      </div>
    </div>
    <div class="te_chapter_body">
      <div>
        <ul>
          <li v-for="(chapter, index) in chapters" :key="index">
            <div class="te_chapter_item w100">
              <div class="w60" @click="showSessions(chapter.id)">
                <h5>Chapter {{ (++index) }} : {{ chapter.name }}</h5>
              </div>
              <div class="w40 txt_right">
                <b-button class="mr-1" variant="primary" @click="onOpenForm('FormSession', true,  chapter)">+</b-button>
                <b-button class="mr-1" variant="info"><b-icon icon="pencil-fill" variant="white" @click="onOpenForm('FormChapter', false, chapter)" /></b-button>
                <b-button variant="danger" @click="onConfirmDelete('chapter', chapter)"><b-icon icon="trash-fill" variant="white" /></b-button>
              </div>
            </div>
            <ul class="te_session_group" :class="{ te_session_active: groupActive.includes(chapter.id) }">
              <li class="te_chapter_session" v-for="(session, index) in chapter.sessions" :key="index">
                <div class="w15 mr-3">
                  <b-embed
                    type="iframe"
                    aspect="16by9"
                    :src="session.url_video"
                    allowfullscreen
                  ></b-embed>
                </div>
                <div class="flex1">
                  <h5>{{ session.name }}</h5>
                </div>
                <div class="w10 txt_right">
                  <b-button class="mb-1" variant="info" @click="onOpenForm('FormSession', false,  session)"><b-icon icon="pencil-fill" variant="white" /></b-button>
                  <br/>
                  <b-button variant="danger"  @click="onConfirmDelete('session', session)"><b-icon icon="trash-fill" variant="white" /></b-button>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { addChapter, updateChapter, deleteChapter } from '../../../api/chapter'
import { addSession, updateSession, deleteSession } from '../../../api/session'
import { getChapterList } from '../../../api/course.js'
import FormChapter from './FormChapter'
import FormSession from './FormSession'
export default {
  props: {
    elmntId: {
      default: null
    }
  },
  components: {
    FormChapter,
    FormSession
  },
  data() {
    return {
      chapters: [],
      groupActive: [],
      isShowForm: false,
      comOpts: {
        comName: 'FormChapter',
        isAdd: true,
        data: {}
      },
      selectedDelete: {}
    }
  },
  async created() {
    this.chapters = await getChapterList(this.elmntId)
  },
  methods: {
    onOpenForm(comName, isAdd, data) {
      this.isShowForm = true
      this.comOpts.comName = comName
      this.comOpts.isAdd = isAdd
      this.comOpts.data = data
      this.handleData(this.comOpts.comName, this.comOpts.isAdd, this.comOpts.data)
    },
    handleData(comName, isAdd, data) {
      if (comName === 'FormChapter') {
        this.comOpts.data = {
          'id': isAdd ? 0 : data.id,
          'name': isAdd ? null : data.name,
          'created_by': 'anhnh',
          'updated_by': 'anhnh'
        }
      } else {
        this.comOpts.data = {
          'id': isAdd ? 0 : data.id,
          'name': isAdd ? null : data.name,
          'about': isAdd ? null : data.about,
          'url_video': isAdd ? null : data.url_video,
          'time': isAdd ? 0 : data.time,
          'chapter_id': isAdd ? data.id : data.chapter_id,
          'created_by': 'anhnh',
          'updated_by': 'anhnh'
        }
      }
    },
    onAddData: async function(data) {
      if (this.comOpts.comName === 'FormChapter') {
        this.onAddChapter(data)
      } else {
        this.onAddSession(data)
      }
    },
    onAddChapter: async function(data) {
      this.$store.commit('SET_LOADING')
      try {
        const body = {
          'name': data.name.trim(),
          'course_id': this.elmntId,
          'created_by': 'anhnh',
          'updated_by': 'anhnh'
        }
        await addChapter(body)
        this.showResAction('success', 'Session successfully created.')
        this.chapters = await getChapterList(this.elmntId)
      } catch (error) {
        this.showResAction('warning', error.response?.data?.message || error.message)
      }
      this.isShowForm = false
      this.$store.commit('SET_DONE_LOADING')
    },
    async onEditData(data) {
      if (this.comOpts.comName === 'FormChapter') {
        this.onEditChapter(data)
      } else {
        this.onEditSession(data)
      }
    },
    async onEditChapter(data) {
      this.$store.commit('SET_LOADING')
      try {
        await updateChapter(data)
        this.showResAction('success', 'Chapter successfully updated.')
        this.chapters = await getChapterList(this.elmntId)
      } catch (error) {
        this.showResAction('warning', error.response?.data?.message || error.message)
      }
      this.isShowForm = false
      this.$store.commit('SET_DONE_LOADING')
    },
    onAddSession: async function(data) {
      try {
        const body = {
          'name': data.name.trim(),
          'url_video': data.url_video,
          'about': data.about,
          'chapter_id': data.chapter_id,
          'time': data.time,
          'created_by': 'anhnh',
          'updated_by': 'anhnh'
        }
        await addSession(body)
        this.showResAction('success', 'Session successfully created.')
        this.chapters = await getChapterList(this.elmntId)
      } catch (error) {
        this.showResAction('danger', error.response?.data?.message || error.message)
      }
      this.isShowForm = false
      this.$store.commit('SET_DONE_LOADING')
    },
    onEditSession: async function(data) {
      this.$store.commit('SET_LOADING')
      try {
        await updateSession(data)
        this.showResAction('success', 'Session successfully updated.')
        this.chapters = await getChapterList(this.elmntId)
      } catch (error) {
        this.showResAction('danger', error.response?.data?.message || error.message)
      }
      this.isShowForm = false
      this.$store.commit('SET_DONE_LOADING')
    },
    closeForm() {
      this.isShowForm = false
      this.comOpts.comName = 'FormChapter'
      this.comOpts.isAdd = true
      this.comOpts.data = {
        id: 0,
        mane: null
      }
    },
    showSessions(chapterId) {
      if (this.groupActive.includes(chapterId)) {
        this.groupActive.splice(this.groupActive.indexOf(chapterId), 1)
      } else {
        this.groupActive.push(chapterId)
      }
    },
    showForm() {
      this.comOpts.comName = 'FormSession'
      this.isShowForm = true
    },
    onConfirmDelete(elmntName, data) {
      this.selectedDelete.elmntName = elmntName
      this.selectedDelete.data = data
      this.$swal({
        title: 'Are you sure?',
        text: "You won't be able to revert this!",
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        confirmButtonText: 'Yes, delete it!'
      }).then((result) => {
        if (result.isConfirmed) {
          this.onDeleteData()
        }
      })
    },
    async onDeleteData() {
      this.$store.commit('SET_LOADING')
      try {
        if (this.selectedDelete.elmntName === 'chapter') {
          await deleteChapter(this.selectedDelete.data.id)
          this.showResAction('success', 'Chapter successfully deleted.')
        } else {
          await deleteSession(this.selectedDelete.data.id)
          this.showResAction('success', 'Session successfully deleted.')
        }
      } catch (error) {
        this.res('danger', error.response?.data?.message || error.message)
      } finally {
        this.chapters = await getChapterList(this.elmntId)
      }
      this.$store.commit('SET_DONE_LOADING')
    }
  }
}
</script>

<style lang="scss">
.te_chapter {
  margin: 0 1em;
}
.te_chapter_heading {
  display: flex;
  align-items: center;
  button {
    font-weight: bold;
  }
}
.te_chapter_body {
  ul {
    list-style-type: none;
    padding-inline-start: 0;
  }
}
.te_chapter_item {
  height: 50px;
  display: flex;
  align-items: center;
  background-color: white;
  padding: 1em;
  border-radius: 0.5em;
  cursor: pointer;
  box-shadow: 3px 3px 0 rgb(0 0 0 / 10%);
  button {
    font-size: 75%;
  }
  margin: 1em 0;
}
.te_chapter_session {
  display: flex;
  align-items: center;
  background-color: wheat;
  margin: 0.5em 0 0.5em 2em;
  padding: 1em;
  border-radius: 0.5em;
  box-shadow: 3px 3px 0 rgb(0 0 0 / 10%);
  button {
    font-size: 50%;
  }
}
.te_session_group {
  display: none;
}
.te_session_active {
  display: block;
}
.te_form_chapter {
  border: 1px solid;
  background-color: white;
  padding: 1em;
  border-radius: 0.5em;
  margin: 1em 0;
  display: none;
}
.te_active_form_chapter {
  display: block;
}
</style>
