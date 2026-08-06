# Display Editor: Grid Row Indexing and Group Label Semantics
Create date: 2026-07-20
Last modified date: 2026-07-20

Notes on two related defects found in the Display Editor (`pgDisplayFields`), the
decisions taken, and the limitations knowingly left in place.

Line numbers refer to `Project_Definition_scripts.txt` as of this date and will drift.

---

## 1. Row number / array index mismatch under sorting

### Symptom

With the grid sorted by Field Name, setting the group on the selected field applied
the group to a *different* field. Example: "Verbatim Locality" was selected at visual
row 2 and assigned to group "Location"; after the refresh, "Source" (`sGridData[2]`)
was the field that had been grouped.

### Cause

`the pgHilitedRows of widget "pgDisplayFields"` returns **visual row numbers in the
widget's current display order**. Every consumer treats that number as an index into
`sGridData`:

- `deAssignToGroup pRow, ...` — `sGridData[pRow]`
- `deMoveField` — builds block/remainder from `sGridData[i]`
- `deRefreshGrid` — selection restore
- `deSetGroup` / `deProcessGroupChoice` — `sTargetRow`

Those two numbering schemes agree only while the widget is in its as-built order.

Header sorting is **not** a PolyGrid feature. It is hand-rolled in the control script
of `pgDisplayFields` (`on headerClick`, ~line 7078). That handler does:

```
put the text of me into tText
sort lines of tText ... by item pColumnNumber of each
set the text of me to tText
```

It re-orders the widget's own text and never touches `sGridData`. From the moment a
header is clicked, visual row *n* and `sGridData[n]` are unrelated.

`deRefreshGrid` then rebuilds the grid from `sGridData` in array order, so the sort
silently disappears at the same moment the wrong field is shown as modified — which is
why the fault looks like a refresh bug rather than an indexing bug.

### Decision: disable header sorting

`headerClick` in the `pgDisplayFields` control script is reduced to a documented no-op
rather than deleted, so the omission reads as deliberate:

```
-- Header sorting disabled 2026-07-20.
-- The sort re-ordered the widget's text only, leaving sGridData untouched,
-- so pgHilitedRows no longer matched the sGridData index. Handlers that take
-- a row number (deAssignToGroup, deMoveField, deRefreshGrid's selection
-- restore) then acted on the wrong field. Re-enable only alongside a
-- visual-row -> array-index translation.
on headerClick pColumnNumber
   -- intentionally does nothing
end headerClick
```

`local sDirection` at the top of that script becomes unused; harmless, and needed again
if sorting is restored.

### If sorting is restored later

The `#` column already holds the true `sGridData` index — `deRefreshGrid` writes the
loop counter `i` there, and it survives the sort. A helper that maps a list of visual
rows to their column-1 values, applied immediately after every `pgHilitedRows` read,
would fix all consumers with a one-line change each.

`deMoveField` is the exception and needs a policy decision, not a translation. "Move
down" in a sorted view has no coherent meaning in terms of `displayOrder`; translating
indices lets it run without error but produces arbitrary placement. It should refuse to
operate while a sort is active, or offer to clear the sort first.

### Not investigated

The same hand-rolled `headerClick` pattern exists on `pgColumnsData`
(~line 4513) and on the Services and Prompts grids. Whether they are affected depends
on whether anything downstream maps their row numbers onto a parallel array.

---

## 2. Group column semantics

### The three states

Group membership is carried by `groupId` alone. `groupLabel` is **not** a membership
flag — it is the heading text drawn above the group in the curation form.
`Metadata_scripts.txt` (~line 1523):

```
-- Create header label only if groupLabel is not empty
if pGroup["groupLabel"] is not empty then
```

`dsGenerateDefaultDisplayDef` (~line 1839) uses that deliberately: it always assigns a
`groupId`, but writes the `groupLabel` only when there is more than one group and the
category is not "General" — a heading over every field in a single-group layout is
pointless clutter.

That yields three legitimate states:

| `groupId` | `groupLabel` | Meaning |
|---|---|---|
| empty | empty | Genuinely ungrouped |
| set | empty | In a group, heading suppressed |
| set | set | In a named group |

### Defect

`deRefreshGrid` (~line 5536) had these inverted:

```
if sGridData[i]["groupLabel"] is not empty then
   put sGridData[i]["groupLabel"] into tGroupLabel
else if sGridData[i]["groupId"] is not empty then
   put "(ungrouped)" into tGroupLabel     -- actually grouped
else
   put "" into tGroupLabel                -- actually ungrouped
end if
```

Grouped-but-unlabelled fields were reported as "(ungrouped)"; genuinely ungrouped
fields showed blank. On a default display this meant nearly every row read
"(ungrouped)" while every field was in fact grouped.

### Fix

- both empty → blank
- `groupId`, no label → show the `groupId` (or `"(group " & groupId & ")"`)
- label present → show the label

Showing the raw `groupId` is mildly technical for the UI, but it is truthful, needs no
invented placeholder string, and distinguishes two unlabelled groups from each other.

The string `"(ungrouped)"` appeared nowhere else in the codebase. The group popup and
`deAssignToGroup` read `groupId`/`groupLabel` from `sGridData` directly, never the
grid's text, so this change is display-only.

---

## 3. Known limitation: one-way removal from an unlabelled group

Not fixed. Recorded because it is a real trap.

`deSetGroup` builds the group popup from labels and skips unlabelled groups outright
(~line 5896):

```
repeat for each key tGId in tGroupLabels
   if tGroupLabels[tGId] is empty then next repeat
   put tGroupLabels[tGId] & cr after tListItems
end repeat
```

So on a default display, where all groups are unlabelled by design:

1. "Remove from Group" clears `groupId`, `groupLabel` and `bgColor` on the field.
2. The field is now the only genuinely ungrouped item on the display.
3. The group it left is unlabelled, so it never appears in the popup.
4. The only route back is "New Group...", which creates a *different* group.

The removal is effectively irreversible through the UI.

A related weakness: `deProcessGroupChoice` resolves the chosen menu item back to a
`groupId` by scanning for the first matching `groupLabel` (~line 5967). Two groups
sharing a label would collide, and the second becomes unreachable.

### Root cause

`groupLabel` does double duty — the group's name *and* the "render a heading?" flag.
Separating them (always store a name, add a `showLabel` boolean) resolves both the
popup exclusion and the label collision, and makes the Group column unambiguous
without needing to display raw ids.

That touches the display-def format, `dsGenerateDefaultDisplayDef`, the Metadata
renderer and the migration of existing `.specimate` files. Deferred.
