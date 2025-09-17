<script lang="ts">
</script>

<button {...$$restProps}>
  <slot />
</button>

<style>
  button {
    background-color: #007bff;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
  }
</style>