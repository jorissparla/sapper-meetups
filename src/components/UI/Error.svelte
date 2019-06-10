<script>
  import Modal from "./Modal.svelte";
  export let message;
</script>

<Modal title="An Error Occurred" on:cancel>
  <p>{message}</p>
</Modal>
