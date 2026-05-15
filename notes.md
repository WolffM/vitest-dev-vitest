## Steps to reproduce
1. Enabled pnpm via corepack and installed dependencies with `pnpm install`.
2. Reproduced Temporal conversion behavior from the unit-test workspace with:
   `cd /home/runner/work/vitest-dev-vitest/vitest-dev-vitest/test/unit && node -e "import('temporal-polyfill').then(({Temporal})=>{for(const v of [Temporal.Instant.from('2020-01-01T00:00:00Z'),Temporal.ZonedDateTime.from('2020-01-01T00:00:00+00:00[UTC]'),Temporal.PlainDate.from('2020-01-01'),Temporal.PlainDateTime.from('2020-01-01T00:00:00')]){try{console.log(v.toString(),'->',new Date(v).toISOString())}catch(e){console.log(v.toString(),'->ERR',e.message)}}})"`.
3. Confirmed that Vitest currently converts non-Date values to `new Date(value)` in `setSystemTime`.

## Observed
Temporal values consistently failed conversion through `new Date(temporalValue)` and printed `ERR Cannot use valueOf` for all tested Temporal classes. This shows the same coercion pitfall that `vi.setSystemTime` currently hits, because its implementation routes non-Date inputs through `new Date(now)`. In practice, this blocks direct use of Temporal objects and requires users to manually convert to epoch milliseconds first.

## Expected
`vi.setSystemTime` should accept Temporal inputs the same way it accepts Date-compatible inputs and should not require a manual workaround. Passing a Temporal value should set the mocked clock correctly, allowing `Date.now()` and related APIs to reflect the intended instant. Type signatures should also allow Temporal arguments so that valid Temporal-based usage compiles without type errors.
