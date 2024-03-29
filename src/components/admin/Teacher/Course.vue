<template>
  <div>
    <div class="container">
      <ul class="te_list_item">
        <li v-for="(course, index) in table.data" :key="index">
          <div class="mr-2">
            <b-img :src="course.url_image" />
          </div>
          <div class="flex1">
            <div>
              <h4>{{ course.name }}</h4>
            </div>
            <div>
              <span class="mr-2"><b>Price:</b> {{ getPrice(course.price) }}</span>
              <span class="mr-2"><b>Views:</b> {{ course.views }}</span>
              <span class="mr-2"><b>Likes:</b> {{ course.likes }}</span>
              <span class="mr-2"><b>Purchases:</b> {{ course.purchases }}</span>
              <br/>
              <span class="mr-2"><b>Created at:</b> {{ convertTSToDateTime(course.created_at) }}</span>
              <span class="mr-2"><b>Updated at:</b> {{ convertTSToDateTime(course.updated_at) }}</span>
            </div>
          </div>
          <div class="w30 txt_right">
            <button class="btn btn-success mr-2" @click="toCourseDetail(course.id)">Approve</button>
            <button class="btn btn-danger mr-2" @click="toCourseDetail(course.id)">Cancel</button>
          </div>
        </li>
      </ul>
      <div v-if="table.total > 5">
        <b-pagination
          v-model="table.page"
          :total-rows="table.total"
          :per-page="table.perPage"
          class="mt-4 float-right mr-4"
          @change="onPageChange"
        >
          <template #first-text><span class="text-success">First</span></template>
          <template #prev-text><span class="text-danger">Prev</span></template>
          <template #next-text><span class="text-warning">Next</span></template>
          <template #last-text><span class="text-info">Last</span></template>
          <template #ellipsis-text>
            <b-spinner small type="grow" />
            <b-spinner small type="grow" />
            <b-spinner small type="grow" />
          </template>
          <template #page="{ page, active }">
            <b v-if="active">{{ page }}</b>
            <i v-else>{{ page }}</i>
          </template>
        </b-pagination>
      </div>
    </div>
  </div>
</template>

<script>
import { deleteCourse } from '../../../api/course'
import { getCourseListOfTeacher } from '../../../api/public'
export default {
  props: {
    elmntId: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      allCourses: [],
      table: {
        data: [],
        total: 0,
        page: 1,
        perPage: 5
      },
      selectedDelete: {}
    }
  },
  created() {
    this.setPageParam(this.$route.query.pageTag)
    this.getDataList(this.table.page)
  },
  methods: {
    setPageParam(page) {
      if (page) this.table.page = Number(page)
      else this.table.page = 1
    },
    async getDataList(page = 1) {
      const params = {
        page: page,
        teacher_id: this.elmntId,
        limit: this.table.perPage,
        keyword: ''
      }

      const { list, total } = await getCourseListOfTeacher(params)
      this.table.data = list
      this.table.total = total
    },
    onPageChange(page) {
      this.getDataList(page)
      this.$router.push({
        query: { ...page !== 1 && { page }}
      })
    },
    toCourseDetail(courseId) {
      this.showInfo('Chapter', courseId)
      this.$router.push({
        name: 'courseDetailTeacherManeger',
        params: { id: courseId }
      })
    },
    onConfirmDelete(data) {
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
        await deleteCourse(this.selectedDelete.data.id)
      } catch (error) {
        this.res('danger', error.response?.data?.message || error.message)
      } finally {
        this.getDataList(this.table.page)
      }
      this.$store.commit('SET_DONE_LOADING')
    }
  }
}
</script>

<style lang="scss">
.te_list_item {
  list-style-type: none;
  padding-inline-start: 0;
  font-size: 18px;
  li {
    background-color: white;
    border-radius: 1em;
    padding: 1em;
    display: flex;
    align-items: center;
    cursor: pointer;
    box-shadow: 3px 3px 0 rgba(0,0,0,0.1);
    margin: 1.5em 0;
  }
  li:hover {
    background-color: wheat;
  }
}
</style>
