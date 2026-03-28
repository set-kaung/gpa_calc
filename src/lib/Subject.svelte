<script lang="ts">
  let {
    id,
    name = $bindable(""),
    credit = $bindable(3),
    grade = $bindable(""),
  } = $props();

  const gradeSymbols = ["A", "A-", "B+", "B", "B-", "C+", "C", "C-", "D", "F"];
</script>

<div class="subject-container">
  <label for="subject-name">Subject</label>
  <input type="text" name="subject-name" id="subject-name" bind:value={name} />
  <label for="credit-amount">Credit</label>
  <input
    type="number"
    name="credit-amount"
    id="credit-amount"
    bind:value={credit}
  />
  <label for="grade">Grade</label>
  <select bind:value={grade}>
    {#each gradeSymbols as symbol}
      <option value={symbol}>
        {symbol}
      </option>
    {/each}
  </select>
</div>

<style>
</style>
