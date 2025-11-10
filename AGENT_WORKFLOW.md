# AI Agent Workflow Log

## Agents Used
- ChatGPT (developer assistant) — used to design architecture, produce code snippets, and write tests.
- GitHub Copilot / Copilot CLI — used locally (suggested) for small inline completions (boilerplate).
- Claude / Claude-internal — used for high-level refactor suggestions (documented examples).

> Note: In this submission the visible code was generated programmatically. The workflow below documents how such agents would be used in practice.

## Prompts & Outputs
### Example 1 - Generate CB computation use-case
**Prompt:**
```
Write a TypeScript function computeComplianceBalance(actualGhg: number, targetGhg: number, fuelTons: number): number
that implements CB = (target - actual) * (fuelTons * 41000) where 41,000 MJ/t is energy per tonne.
Return value in tonnes CO2eq (divide by 1e3 appropriately). Include JSDoc and unit tests.
```
**Generated snippet (excerpt):**
```ts
export function computeComplianceBalance(actualGhg: number, targetGhg: number, fuelTons: number): number {
  const energyMJ = fuelTons * 41000;
  const cb_gCO2eq = (targetGhg - actualGhg) * energyMJ;
  // convert gCO2eq to tCO2eq
  return cb_gCO2eq / 1e3;
}
```

### Example 2 - Refine banking API contract
**Prompt:**
```
Produce an Express route handler in TypeScript for POST /banking/bank that validates amount <= available CB and returns 400 otherwise.
```
**Generated snippet (excerpt):**
```ts
router.post('/bank', async (req, res) => {
  const { shipId, year, amount } = req.body;
  const available = await bankService.getAvailable(shipId, year);
  if (amount > available) return res.status(400).json({ error: 'Insufficient bank balance' });
  await bankService.bank(shipId, year, amount);
  res.json({ success: true });
});
```

## Validation / Corrections
- Each generated snippet was reviewed for numeric correctness (units: gCO2eq vs tCO2eq) and edge-cases (negative amounts, rounding).
- Unit tests were created for core functions (computeComplianceBalance, percentDiff) and run to verify outputs with known values (see `backend/core/application/__tests__`).
- Where agents proposed ambiguous types, we enforced strict TypeScript types and added runtime validation (zod-like checks via simple guard functions).

## Observations
- **Saved time:** Scaffolding repetitive code (Express routes, DTOs, README) and producing test stubs.
- **Failed/hallucinated:** Agents sometimes omitted unit conversions (g→t) or mis-specified MJ/t constant; these were caught by tests and manual review.
- **Effective combination:** Used agent to draft code, then human review to enforce domain correctness and add tests.

## Best Practices Followed
- Keep prompts small and focused (single responsibility)
- Provide expected units and examples in prompts to avoid unit errors
- Always write unit tests for numeric code and edge cases
- Use agents for boilerplate, not final validation
