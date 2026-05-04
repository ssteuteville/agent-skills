# Time Windows

Rules for resolving the "yesterday" window in Step 1 of [../SKILL.md](../SKILL.md). Today's date comes from the system clock — convert to absolute ISO before constructing any JQL, `gh search`, or `git log` query.

## Default rules

| Day the user runs the skill | Window start | Window end | Confirm with user? |
|---|---|---|---|
| Tuesday | yesterday 00:00 local | now | No |
| Wednesday | yesterday 00:00 local | now | No |
| Thursday | yesterday 00:00 local | now | No |
| Friday | yesterday 00:00 local | now | No |
| Monday | previous Friday 09:00 local | now | No (auto-extend) |

The Monday auto-extend is the user's chosen default — do not prompt.

## Holiday or PTO extension

The default Monday rule already covers the standard 2-day Fri→Mon gap. The extension rules below only apply when the gap is longer than that — e.g. Tuesday after a 3-day holiday weekend, or the first day back from PTO.

- **Gap of 3 calendar days** (Tue after a Mon holiday): auto-extend to 09:00 local on the first day before the gap. No prompt.
- **Gap > 3 calendar days** (coming back from PTO, longer holiday breaks): auto-detect the gap, propose the extended window to the user, **wait for confirmation** before issuing any queries. The proposed start should be 09:00 local on the first business day before the gap.

Holiday detection is best-effort — there is no built-in calendar of US holidays. Infer the gap from the system date relative to the user's last activity if available; otherwise prompt with a proposed window and let the user confirm or adjust.

## Output formats

The same window start gets formatted differently per tool:

- `gh search ... --committer-date=">=YYYY-MM-DD"` and `--updated=">=YYYY-MM-DD"` — date only, no time. Use the date of the window start; the search is inclusive.
- JQL `status changed DURING ("YYYY-MM-DD HH:mm", "YYYY-MM-DD HH:mm")` — both bounds, with `HH:mm` precision. The `DURING` clause requires the exact double-quoted format.
- `git log --since="YYYY-MM-DDTHH:MM:SS"` — ISO 8601 with `T` separator. Local time is fine; git interprets it in the local timezone.
- Slack search `after:YYYY-MM-DD` — date only.

## Edge cases

- **Skill run before 09:00 local on a non-Monday.** The "24h ending now" window will include items from the day before yesterday. That's usually fine; surface the window in the rendered output if it spans more than one calendar day.
- **Skill run very late (after 22:00 local).** The window still ends at "now" — items from earlier today are correctly counted as Today, not Yesterday.
- **User in a non-Pacific timezone.** Use the system's local timezone consistently; don't try to translate to SkySlope HQ time. The user knows what their own day looked like.
