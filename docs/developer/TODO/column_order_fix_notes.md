# smArrayToOrderList Column Order Fix
Create date: 2026-07-29
Last modified date: 2026-07-29

## Problem

Exported dataset columns appear in arbitrary order, and that order changes between
sessions. The same fault also scrambles the Columns field in the OCR window and the
display picker menus.

## Root cause

`smArrayToOrderList` (OCR stack, line 2656) builds its output with:

```
repeat for each key tKey in pArray
```

LiveCode gives no ordering guarantee for `repeat for each key`, so the sequence is
effectively arbitrary and can differ from run to run.

This is the same fault previously fixed in `smProject.loadDefinitionToUI` by switching
to `smOrderedColumnKeys`. That fix was applied at the one call site; the shared
function was left as it was, so every other caller is still affected.

## Affected call sites

| Location | Call | Impact |
|---|---|---|
| OCR 425 | `smArrayToOrderList(tCols, comma, "name")` | `smGetOutputColumns()` fallback - drives export column order |
| projects 468 | `smArrayToOrderList(tOrder, comma)` | `smProject.get.outputOrder()` when stored as an array |
| Project_Definition 300 | `smArrayToOrderList(tDataA, comma)` | `dsParseOutput` - outputOrder parsed from a TSV template |
| Metadata 1937 | `smArrayToOrderList(tDisplaysA, cr, "name")` | display picker menu |
| OCR 4368 | `smArrayToOrderList(tDisplaysA, CR, "name")` | display picker menu |
| Project_Definition 3113 | `smArrayToOrderList(tDisplaysA, cr, "name")` | display picker menu |

Downstream of `smGetOutputColumns()`:

- OCR 8323 - "save extracted entities" button, TSV header and column order
- OCR 8346 - "Export data" button, TSV header and column order
- OCR 8252 - `smOpenTable.NER`
- Prompts 226, Tabular 269 - outputOrder field population

## Fix

Two changes, both in the OCR stack script alongside `smOrderedColumnKeys`.

### 1. New helper: deterministic key ordering

```livecode
-- Returns the keys of pArray in a deterministic order:
--   1. by each element's "order" property, when present
--   2. otherwise by key - numerically when every key is a number, else alphabetically
-- Never relies on "repeat for each key" iteration order to sequence output.
function smOrderedArrayKeys pArray
   local tKey, tKeys, tHasOrder, tAllNumeric

   if pArray is not an array then return empty

   put false into tHasOrder
   put true into tAllNumeric
   repeat for each key tKey in pArray
      if tKey is not a number then put false into tAllNumeric
      if pArray[tKey] is an array and pArray[tKey]["order"] is not empty then
         put true into tHasOrder
      end if
   end repeat

   if tHasOrder then return smOrderedColumnKeys(pArray)

   put the keys of pArray into tKeys
   if tAllNumeric then
      sort lines of tKeys ascending numeric
   else
      sort lines of tKeys ascending text
   end if

   return tKeys
end smOrderedArrayKeys
```

### 2. Rewritten smArrayToOrderList

```livecode
function smArrayToOrderList pArray, pDel, pItem
   -- Convert an array to a delimited list in a deterministic order.
   -- 2026-07-29 was "repeat for each key", which gives arbitrary order.
   local tKeys, tKey, tOrder, tOldDel, i

   if pArray is not an array then return empty

   put smOrderedArrayKeys(pArray) into tKeys
   if tKeys is empty then return empty

   put the itemDel into tOldDel
   set the itemDelimiter to pDel

   put 0 into i
   repeat for each line tKey in tKeys
      add 1 to i
      if pItem is not empty then
         put pArray[tKey][pItem] into item i of tOrder
      else
         put pArray[tKey] into item i of tOrder
      end if
   end repeat

   set the itemDel to tOldDel
   return tOrder
end smArrayToOrderList
```

The signature, parameters and return value are unchanged, so no call site needs
editing.

Incidental: the original declared `local tOrder, i, tNum` but used an undeclared
`tDelOld`, and `tNum` was unused. Both tidied above.

## What each caller gets

- **Definition columns** (OCR 425) carry an `"order"` property, so they route to
  `smOrderedColumnKeys` and come back in definition order. This is the one that
  matters for exports.
- **outputOrder as an array** (projects 468) and **TSV-parsed columns**
  (Project_Definition 300) are numerically keyed, so they sort numerically into
  their original sequence.
- **Display menus** (3 sites) have no `"order"` property and non-numeric keys, so
  they sort by `displayId` - stable between sessions, but not meaningful to a user.
  If alphabetical by name is wanted, sort at the call site rather than changing the
  shared function:

  ```livecode
  put smArrayToOrderList(tDisplaysA, cr, "name") into tMenu
  sort lines of tMenu
  ```

## Edge cases

- **Part-migrated definitions** where some columns have `"order"` and some do not:
  `smOrderedColumnKeys` falls back to the key as the sort value, and a non-numeric
  key sorts as 0. Those elements keep an arbitrary order relative to each other.
  Worth a validation pass on definitions rather than more code here.
- **Values containing the delimiter** still corrupt the list. Pre-existing, not
  addressed.

## Once this is in

The Tabular card's `pColumns` parameter hack becomes optional - `smGetOutputColumns()`
would then return a correct, stable order. Leaving the hack in place is harmless:
`smTableColumns` prefers `pColumns`, then the record's `entities`, then the OCR
Columns field. Removing it later means reverting four small edits.

## Separate bug found while tracing this

OCR 8317, "save extracted entities" button: the TSV is assembled into `tEntities`,
but the final line writes a variable that is never assigned:

```livecode
   smWriteDataFile.UTF8 tOutput, tFile
```

Should be `tEntities`. As it stands the button writes an empty file.

## Test checklist

1. Export a dataset, note the column order, quit and relaunch, export again - the
   order should be identical and should match the definition.
2. Confirm the OCR window Columns field keeps its order across a project reload.
3. Open the display picker on all three screens and confirm it still populates.
4. Load a record into the Tabular screen and confirm columns are unaffected.
