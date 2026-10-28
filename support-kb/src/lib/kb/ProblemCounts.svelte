<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Cause, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  // One id, one report (the export pages overlap), then only what was merged into this ticket.
  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));

  // The desk's own view: a report counts under the cause of its tenant (causes.ts), even when the
  // desk read it as a one-day move (desk-notes.json). Those are footnoted, not counted apart.
  const under = (c: Cause) => merged.filter((r) => causeOf(r, tenantOf(r)) === c);
  const oneDay = (c: Cause) => under(c).filter((r) => notes.oneDayMove.includes(r.id)).length;

  const rows = $derived([
    { problem: 'parser reads dates month first', cause: 'parser' as Cause, shows: 'day and month swapped, any channel' },
    { problem: 'booking-v3 form hint says MM/DD/YYYY', cause: 'hint' as Cause, shows: 'day and month swapped, form channel only' },
  ].map((row) => ({ ...row, count: under(row.cause).length, oneDay: oneDay(row.cause) })));
  const total = $derived(rows.reduce((n, row) => n + row.count, 0));
</script>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>what people see</th></tr></thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr>
        <td>{row.problem}</td>
        <td>{row.count}</td>
        <td>{row.shows}{#if row.oneDay}{` · ${row.oneDay} read as a one-day move`}{/if}</td>
      </tr>
    {/each}
  </tbody>
  <tfoot>
    <tr class:off={total !== merged.length}>
      <td>both</td><td>{total}</td><td>{total === merged.length ? `matches the ${merged.length} merged into #${ticket}` : `does not match the ${merged.length} merged into #${ticket}`}</td>
    </tr>
  </tfoot>
</table>

<style>
  tfoot td { font-weight: 600; }
  .off td { color: var(--warn); }
</style>
