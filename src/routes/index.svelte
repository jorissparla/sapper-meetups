<script context="module">
  export function preload(page) {
    console.log(page);
    return this.fetch(
      "https://svelte-meetups-1c01f.firebaseio.com/meetups.json"
    )
      .then(res => {
        if (!res.ok) {
          throw new Error(" Invalid request");
        }
        return res.json();
      })
      .then(data => {
        const loadedMeetups = [];
        for (const key in data) {
          loadedMeetups.push({ id: key, ...data[key] });
        }
        return { fetchedMeetups: loadedMeetups.reverse() };
      })
      .catch(err => {
        error = err;
        this.error("Could not fetch meetups");
        console.log(error);
      });
  }
</script>

<script>
  import { createEventDispatcher, onMount, onDestroy } from "svelte";
  import { fade, fly, scale } from "svelte/transition";
  import { flip } from "svelte/animate";
  const dispatch = createEventDispatcher();
  import EditMeetup from "../components/Meetups/EditMeetup.svelte";
  import LoadingSpinner from "../components/UI/LoadingSpinner.svelte";
  import MeetupItem from "../components/Meetups/MeetupItem.svelte";
  import MeetupFilter from "../components/Meetups/MeetupFilter.svelte";
  import Button from "../components/UI/Button.svelte";
  import meetups from "../store/meetups-store.js";
  let editMode = false;
  let editedId = null;
  let isLoading = false;

  let favsOnly = false;
  export let fetchedMeetups = [];
  let loadedMeetups = [];
  let unsubscribe;

  $: filteredMeetups = favsOnly
    ? loadedMeetups.filter(m => m.isFavorite)
    : loadedMeetups;

  onMount(() => {
    unsubscribe = meetups.subscribe(items => {
      loadedMeetups = items;
    });
    meetups.setMeetups(fetchedMeetups);
  });

  onDestroy(() => {
    if (unsubscribe) {
      unsubscribe();
    }
  });
  function setFilter(event) {
    favsOnly = event.detail === 1;
  }

  function savedMeetup(event) {
    editMode = false;
    editedId = null;
  }

  function cancelEdit() {
    editMode = false;
    editedId = null;
  }

  function startEdit(event) {
    editMode = true;
    editedId = event.detail;
  }
  function startAdd() {
    editMode = true;
  }
</script>

<style>
  #meetups {
    width: 100%;
    display: grid;
    grid-template-columns: 1fr;
    grid-gap: 1rem;
  }
  #meetup-controls {
    margin: 1rem;
    display: flex;
    justify-content: space-between;
  }
  #no-meetups {
    margin: 1rem;
  }
  @media (min-width: 768px) {
    #meetups {
      grid-template-columns: repeat(2, 1fr);
    }
  }
</style>

<svelte:head>
  <title>All Meetups</title>
</svelte:head>
{#if editMode}
  <!-- content here -->
  <EditMeetup on:save={savedMeetup} on:cancel={cancelEdit} id={editedId} />
{/if}

{#if isLoading}
  <!-- content here -->
  <LoadingSpinner />
{:else}
  <section id="meetup-controls">

    <MeetupFilter on:select={setFilter} />
    <Button on:click={startAdd}>New Meetup</Button>
  </section>
  {#if filteredMeetups.length === 0}
    <!-- content here -->
    <p id="no-meetups">No Meetups found..</p>
  {/if}
  <section id="meetups">
    {#each filteredMeetups as meetup (meetup.id)}
      <div transition:scale animate:flip={{ duration: 500 }}>
        <MeetupItem
          id={meetup.id}
          imageURL={meetup.imageURL}
          title={meetup.title}
          subtitle={meetup.subtitle}
          description={meetup.description}
          address={meetup.address}
          email={meetup.contactEmail}
          isFavorite={meetup.isFavorite}
          on:showDetails
          on:edit={startEdit} />
      </div>
    {/each}
  </section>

{/if}
