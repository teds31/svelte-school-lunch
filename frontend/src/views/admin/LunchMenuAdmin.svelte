<script>
  import { onMount } from 'svelte'
  import { navigateTo } from 'svelte-router-spa'
  import axios from 'axios'
  import Icon from 'svelte-awesome'
  import { refresh, times } from 'svelte-awesome/icons'
  import { user } from '../../stores.js'

  let lunchWeekList = []
  let loading = true
  let showDeleteModal = false
  let weekToDelete = {}

  let showCreateModal = false
let createWeekOfDate = null

const openCreateModal = () => {
  showCreateModal = true
}

const createLunchWeek = async () => {
  showCreateModal = false
  let newLunchWeek = {
    weekOf: createWeekOfDate, // createWeekOfDate will contain the input from the user
    isPublished: false,
  }
  try {
    loading = true

    // since this is a POST, we need to send a lunchWeek object as the body of the request
    const response = await axios.post(`${process.env.API_ROOT}/api/lunch-week`, newLunchWeek)
    const lunchWeekId = response.data.lunchWeekId
    
    // populate the newLunchWeek with the id from the server response
    newLunchWeek.lunchWeekId = lunchWeekId

    // push the result into lunchWeek list, so that
    // Svelte will update the table
    lunchWeekList.push(newLunchWeek)
    loading = false
  } catch (e) {
    loading = false
    console.error(e)
  }
}

  onMount(async () => {
    try {
      const response = await axios.get(`${process.env.API_ROOT}/api/lunch-week`)
      lunchWeekList = response.data
      loading = false
    } catch (e) {
      console.error(e)
    }
  })

  const openLunchWeekDetails = lunchWeek => {
    const route = `/admin/manage-menus/week-details/${lunchWeek.lunchWeekId}`
    navigateTo(route)
  }

  const openDeleteModal = lunchWeek => {
    weekToDelete = lunchWeek
    showDeleteModal = true
  }

  const deleteLunchWeek = async lunchWeek => {
    showDeleteModal = false
    const lunchWeekId = lunchWeek.lunchWeekId
    try {
      loading = true
      await axios.delete(`${process.env.API_ROOT}/api/lunch-week/${lunchWeekId}`)
      const deletedIndex = lunchWeekList.findIndex(x => x.lunchWeekId === lunchWeekId)
      lunchWeekList.splice(deletedIndex, 1)
      loading = false
    } catch (e) {
      loading = false
      console.error(e)
    }
  }
</script>

<style>
  .clickable {
    cursor: pointer;
  }
</style>

<div>
  <nav class="breadcrumb" aria-label="breadcrumbs">
    <ul>
      <li>
        <a href="/">Home</a>
      </li>
      <li>
        <a href="/admin/manage-menus">Lunch Menu Administration</a>
      </li>
      <li class="is-active">
        <a href="/#">{$user.schoolName}</a>
      </li>
    </ul>
  </nav>
  {#if loading}
    <div class="section">
      <Icon spin data="{refresh}" scale="3" />
    </div>
  {:else}
    <table class="table">
      <thead>
        <tr>
          <th>Week Of</th>
          <th>Published</th>
          <th></th>
        </tr>
      </thead>
      {#each lunchWeekList as lunchWeek}
        <tr>
          <td class="has-text-link clickable" on:click="{() => openLunchWeekDetails(lunchWeek)}">{lunchWeek.weekOf}</td>
          <td>{lunchWeek.isPublished}</td>
          <td class="has-text-grey-light clickable" on:click="{() => openDeleteModal(lunchWeek)}">
            <Icon style="margin-top: 4px;" data="{times}" />
          </td>
        </tr>
      {/each}
    </table>
  {/if}
</div>

<div class="{showDeleteModal ? 'modal is-active' : 'modal'}">
  <div class="modal-background"></div>
  <div class="modal-card">
    <header class="modal-card-head">
      <p class="modal-card-title">Warning</p>
      <button class="delete" on:click="{() => (showDeleteModal = false)}" aria-label="close"></button>
    </header>
    <section class="modal-card-body">Delete Week of {weekToDelete.weekOf}?</section>
    <footer class="modal-card-foot">
      <button class="button is-success" on:click="{() => deleteLunchWeek(weekToDelete)}">Continue</button>
      <button class="button" on:click="{() => (showDeleteModal = false)}">Cancel</button>
    </footer>
  </div>
</div>

<div class="{showCreateModal ? 'modal is-active' : 'modal'}">
  <div class="modal-background"></div>
  <div class="modal-card">
    <header class="modal-card-head">
      <p class="modal-card-title">Create Lunch Week</p>
      <button class="delete" on:click="{() => (showCreateModal = false)}" aria-label="close"></button>
    </header>
    <section class="modal-card-body">
      <div class="field">
        <label class="label">Week Of</label>
        <div class="control">
          <!-- bind users input for the Week Of Date to the createWeekOfDate state var -->
          <input bind:value="{createWeekOfDate}" type="date" class="input" placeholder="yyyy-mm-dd" />
        </div>
      </div>
    </section>
    <footer class="modal-card-foot">
      <button class="button is-success" on:click="{() => createLunchWeek()}">Continue</button>
      <button class="button" on:click="{() => (showCreateModal = false)}">Cancel</button>
    </footer>
  </div>
</div>