<script context="module">
  export function preload(page) {
    console.log(page);
    const meetupId = page.params.id;
    return this.fetch(
      `https://svelte-meetups-1c01f.firebaseio.com/meetups/${meetupId}.json`
    )
      .then(res => {
        if (!res.ok) {
          throw new Error(" Invalid request");
        }
        return res.json();
      })
      .then(data => {
        return { loadedMeetup: { id: meetupId, ...data } };
      })
      .catch(err => {
        error = err;
        this.error(404, "Could not fetch meetups");
        console.log(error);
      });
  }
</script>

<script>
  import { onDestroy, createEventDispatcher } from "svelte";
  import Button from "../components/UI/Button.svelte";
  import meetups from "../store/meetups-store.js";
  const dispatch = createEventDispatcher();
  export let id;
  export let loadedMeetup;
  console.log(loadedMeetup);

  const {
    title,
    subtitle,
    description,
    address,
    imageURL,
    contactEmail,
    isFavorite
  } = loadedMeetup;
</script>

<style>
  section {
    margin-top: 4rem;
  }

  .image {
    width: 100%;
    height: 25rem;
  }

  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .image {
    background: #e7e7e7;
  }

  .content {
    text-align: center;
    width: 80%;
    margin: auto;
  }

  h1 {
    font-size: 2rem;
    font-family: "Roboto Slab", sans-serif;
    margin: 0.5rem 0;
  }

  h2 {
    font-size: 1.25rem;
    color: #6b6b6b;
  }

  p {
    font-size: 1.5rem;
  }
</style>

<section>
  <div class="image">
    <img src={imageURL} alt={title} />
  </div>
  <div class="content">
    <h1>{title}</h1>
    <h2>{subtitle}</h2>
    <p>{description}</p>
    <Button href="mailto:{contactEmail}">Contact</Button>
    <Button type="button" mode="outline" href="/">Close</Button>
  </div>
</section>
