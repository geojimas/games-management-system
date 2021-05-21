<template>
  <div class="container">
    <div v-if="movies.length > 0" class="animate__animated animate__zoomIn">
      <h3 class="title is-3">Create new Game</h3>
      <form @submit.prevent="postAction()" action="POST">
        <div class="field">
          <input
            type="text"
            name="title"
            v-model="newTitle"
            placeholder="title"
            class="input is-primary"
            required
          />
        </div>
        <div class="field">
          <input
            type="text"
            name="kind"
            v-model="newKind"
            placeholder="kind"
            class="input is-primary"
            required
          />
        </div>
        <div class="field">
          <input
            type="number"
            name="reselase_date"
            v-model="newRelease_date"
            placeholder="release date"
            class="input is-primary"
            required
          />
        </div>
        <div class="field">
          <input
            type="text"
            name="director"
            v-model="newdirector"
            placeholder="director"
            class="input is-primary"
            required
          />
        </div>
        <div class="field">
          <input
            type="text"
            name="rate"
            v-model="newRate"
            placeholder="rate"
            class="input is-primary"
            required
          />
        </div>
        <div class="control">
          <button class="button is-primary" type="submit">
            <i class="fas fa-plus-circle"></i>Submit
          </button>
          <h1 class="title is-4" v-if="posting">Posting...</h1>
        </div>
      </form>
    </div>

    <div v-else>
      <h1 class="title is-1">Sorry, No data found :(</h1>
    </div>

    <div class="games animate__animated animate__flipInX">
      <h1 class="title is-1" v-if="loading">Loading...</h1>
      <table
        class="table is-bordered is-striped is-narrow is-hoverable is-fullwidth"
        v-if="movies.length > 0"
      >
        <thead>
          <tr>
            <th>id</th>
            <th>Title</th>
            <th>Director</th>
            <th>Release Date</th>
            <th>Rate</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="movie in movies" v-bind:key="movie._id">
            <td>{{ movie._id }}</td>
            <td>{{ movie.title }}</td>
            <td>{{ movie.director }}</td>
            <td>{{ movie.release_date }}</td>
            <td>{{ movie.rate }}</td>
            <td>
              <div class="action">
                <button class="button is-info" @click="edit(movie)" v-if="movie._id">
                  <i class="far fa-edit"></i>Edit
                </button>

                <button class="button is-danger" @click="deleteAction(movie._id)" v-if="movie._id">
                  <i class="far fa-trash-alt"></i>Delete
                </button>
              </div>
            </td>
          </tr>
        </tbody>
        <h1 class="title is-4" v-if="deleting">Deleting...</h1>
      </table>
    </div>
  </div>
</template>

<script>
import Swal from 'sweetalert2'
import { onBeforeMount, ref, computed } from 'vue'
import axios from 'axios'

export default {
  name: 'Content',
  setup() {
    const movies = ref([])
    const loading = ref(false)
    const posting = ref(false)
    const deleting = ref(false)
    const newTitle = ref('')
    const newKind = ref('')
    const newRelease_date = ref(null)
    const newdirector = ref('')
    const newRate = ref(null)
    const payloadData = ref('')

    // Compute mode for add form and Edit form
    const mode = computed(() => (payloadData.value ? 'Edit' : 'Add'))

    // Edit Button
    const edit = (payload) => {
      if (payload !== false) {
        newTitle.value = payload.title
        newKind.value = payload.kind
        newRelease_date.value = payload.release_date
        newdirector.value = payload.director
        newRate.value = payload.rate
        // Toggle to 'Add' and 'Edit' mode
        payloadData.value = payload
      }
    }

    //life cycle
    onBeforeMount(() => {
      //get Method
      loading.value = true
      axios
        .get('http://localhost:5000/api/movies')
        .then((response) => {
          movies.value = response.data.data
          loading.value = false
        })
        .catch((error) => {
          loading.value = false
          Swal.fire({
            title: 'error!',
            text: error,
            icon: 'error',
            confirmButtonText: 'OK',
          })
        })
    })

    //post method
    const postAction = () => {
      if (mode.value === 'Add') {
        posting.value = true
        axios
          .post('http://localhost:5000/api/movies', {
            title: newTitle.value,
            kind: newKind.value,
            release_date: parseInt(newRelease_date.value),
            director: newdirector.value,
            rate: parseFloat(newRate.value),
          })
          .then((response) => {
            movies.value.push(response.data.data)
            posting.value = false
            newTitle.value = ''
            newKind.value = ''
            newRelease_date.value = null
            newdirector.value = ''
            newRate.value = null
            Swal.fire({
              title: 'Success!',
              text: 'Movie added Successfully!',
              icon: 'success',
              confirmButtonText: 'OK',
            })
          })
          .catch((error) => {
            movies.value = [{ title: 'Posting failed' }]
            Swal.fire({
              title: 'error!',
              text: error,
              icon: 'error',
              confirmButtonText: 'OK',
            })
          })
      } else {
        posting.value = true
        axios
          .patch(`http://localhost:5000/api/movies/${payloadData.value._id}`, {
            title: newTitle.value,
            kind: newKind.value,
            release_date: parseInt(newRelease_date.value),
            director: newdirector.value,
            rate: parseFloat(newRate.value),
          })
          .then((response) => {
            let index = movies.value.map((movie) => movie._id).indexOf(payloadData.value._id)
            movies.value[index] = response.data.data
            posting.value = false
            newTitle.value = ''
            newKind.value = ''
            newRelease_date.value = null
            newdirector.value = ''
            newRate.value = null
            payloadData.value = null
            Swal.fire({
              title: 'Success!',
              text: 'Movie updated Successfully!',
              icon: 'success',
              confirmButtonText: 'OK',
            })
          })
          .catch((error) => {
            movies.value = [{ title: 'Update failed' }]
            Swal.fire({
              title: 'error!',
              text: error,
              icon: 'error',
              confirmButtonText: 'OK',
            })
          })
      }
    }

    // Delete method
    const deleteAction = (_id) => {
      deleting.value = true
      axios
        .delete(`http://localhost:5000/api/movies/${_id}`)
        .then(() => {
          let index = movies.value.map((movie) => movie._id).indexOf(_id)
          movies.value.splice(index, 1)
          deleting.value = false
          Swal.fire({
            title: 'Success!',
            text: 'Movie deleted Successfully!',
            icon: 'success',
            confirmButtonText: 'OK',
          })
        })
        .catch((error) => {
          Swal.fire({
            title: 'error!',
            text: error,
            icon: 'error',
            confirmButtonText: 'OK',
          })
        })
    }

    return {
      movies,
      loading,
      posting,
      deleting,
      newTitle,
      newKind,
      newRelease_date,
      newdirector,
      newRate,
      postAction,
      deleteAction,
      edit,
    }
  },
}
</script>
