<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Cause, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  // One id, one report (the export pages overlap), then only what was merged into this ticket.
  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));

  // Reports the desk read as a one-day move (desk-notes.json) are not a month/day swap, whatever
  // their tenant says. They are their own problem, so they leave the cause they were filed under.
  const isOneDay = (r: Report) => notes.oneDayMove.includes(r.id);
  const swapped = (c: Cause) => merged.filter((r) => !isOneDay(r) && causeOf(r, tenantOf(r)) === c);

  const rows = $derived([
    { problem: 'parser reads dates month first', count: swapped('parser').length, shows: 'day and month swapped, any channel' },
    { problem: 'booking-v3 form hint says MM/DD/YYYY', count: swapped('hint').length, shows: 'day and month swapped, form channel only' },
    { problem: 'date moves by one day', count: merged.filter(isOneDay).length, shows: 'same month, one day off; cause not confirmed' },
  ]);
  const total = $derived(rows.reduce((n, row) => n + row.count, 0));
</script>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>what people see</th></tr></thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr><td>{row.problem}</td><td>{row.count}</td><td>{row.shows}</td></tr>
    {/each}
  </tbody>
  <tfoot>
    <tr class:off={total !== merged.length}>
      <td>all three</td><td>{total}</td><td>{total === merged.length ? `matches the ${merged.length} merged into #${ticket}` : `does not match the ${merged.length} merged into #${ticket}`}</td>
    </tr>
  </tfoot>
</table>

<style>
  tfoot td { font-weight: 600; }
  .off td { color: var(--warn); }
</style>
