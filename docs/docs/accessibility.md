# Accessibility

TableView provides first-class accessibility support through UI Automation (UIA) peers for all its interactive elements. Screen readers, keyboard-only users, and accessibility testing tools can all navigate and interact with the control.

## Supported UI Automation Patterns

| Element | UIA Patterns |
|---------|-------------|
| **TableView** | `GridPattern`, `TablePattern`, `SelectionPattern` (inherited) |
| **TableViewRow** | `SelectionItemPattern` (inherited), `ExpandCollapsePattern` (when row details are available and `RowDetailsVisibilityMode` is `VisibleWhenExpanded`) |
| **TableViewCell** | `GridItemPattern`, `TableItemPattern`, `SelectionItemPattern` |
| **TableViewColumnHeader** | `InvokePattern` (when `CanSort` is `true`) |
| **TableViewRowHeader** | _(structural header)_ |

### GridPattern / TablePattern (TableView)

`TableView` exposes `IGridProvider` and `ITableProvider`:

- **`RowCount`** — number of data rows (items)
- **`ColumnCount`** — number of visible columns
- **`GetItem(row, column)`** — returns the automation element for the specified cell (only for realized/visible cells)
- **`GetColumnHeaders()`** — returns automation elements for all visible column headers
- **`RowOrColumnMajor`** — `RowMajor`

### GridItemPattern / TableItemPattern (Cells)

Each `TableViewCell` exposes:

- **`Row`** — zero-based row index
- **`Column`** — zero-based column index
- **`RowSpan` / `ColumnSpan`** — always 1
- **`ContainingGrid`** — the owning `TableView`
- **`GetColumnHeaderItems()`** — the column header for this cell

### SelectionItemPattern (Cells and Rows)

- `IsSelected` reflects the current selection state
- `Select()`, `AddToSelection()`, `RemoveFromSelection()` manipulate selection
- `SelectionContainer` returns the owning `TableView`
- Row selection through `ListViewItemAutomationPeer` is provided automatically by the base `ListView` infrastructure

### InvokePattern (Column Headers)

When a column is sortable (`CanSort = true`), its header exposes `IInvokeProvider`. Invoking the header cycles sort direction: ascending → descending → unsorted.

### ExpandCollapsePattern (Row Details)

When `RowDetailsVisibilityMode` is `VisibleWhenExpanded`, each row exposes `IExpandCollapseProvider` allowing programmatic expand/collapse of its details pane.

## Automation Names

### TableView
Uses `AutomationProperties.Name` if set; otherwise the name is inherited from the base `ListView` peer.

### Column Headers
Format: `"{headerText}, {sortState}, {filterState}"` where sort and filter suffixes are omitted when not applicable.

Examples:
- `"Name"` — unsorted, unfiltered
- `"Name, Sort Ascending"` — sorted ascending
- `"Name, Sort Descending, Filtered"` — sorted descending and filtered

### Rows
Format: `"Row {N}"` (1-based), or the value of `AutomationProperties.Name` if set.

The help text contains a summary of all cells: `"Name: Waheed Ahmad, Age: 30, City: London"`.

### Cells
Format: `"{columnHeader}, Row {N}, {cellValue}"`.

Examples:
- `"Name, Row 3, Waheed Ahmad"`
- `"Salary, Row 7, 95000"`

Override by setting `AutomationProperties.Name` on the cell's `CellStyle`:

```xml
<TableView.CellStyle>
    <Style TargetType="tvs:TableViewCell">
        <Setter Property="AutomationProperties.Name" Value="Custom Cell Name" />
    </Style>
</TableView.CellStyle>
```

### Row Headers
Uses `AutomationProperties.Name` if set, otherwise the content of the row header as a string, or falls back to `"Row {N}"`.

## Keyboard Navigation

TableView supports full keyboard navigation without breaking any existing behavior:

| Key | Action |
|-----|--------|
| **Arrow Keys** | Move current cell / row |
| **Tab / Shift+Tab** | Move to next/previous cell |
| **Enter** | Move to next row cell (or commit edit) |
| **Space** | Select/deselect current cell or row |
| **F2** | Begin editing the current cell |
| **Escape** | Cancel editing |
| **Home / End** | Move to first/last column in row |
| **Ctrl+Home / Ctrl+End** | Move to first/last row |
| **Page Up / Page Down** | Move by one page |
| **Ctrl+A** | Select all |
| **Ctrl+Shift+A** | Deselect all |
| **Ctrl+C** | Copy to clipboard |

## Screen Reader Behavior

When navigating with a screen reader (e.g. Narrator on Windows):

1. **TableView** announces itself as a "table view" with row and column counts.
2. Moving between **rows** announces the row number and a summary of its content.
3. Moving between **cells** announces the column header, row number, and current cell value.
4. **Column headers** announce their header text along with the current sort state and filter state.
5. **Editing**: When a cell enters edit mode, focus moves to the editing control (e.g. a `TextBox`), which announces itself to the screen reader independently with its own `IValueProvider`.
6. **Selection changes** are announced through `SelectionItemPattern.IsSelected`.

## How Developers Can Improve Automation Names

### Custom Cell Names via Style

```xml
<tvs:TableViewTextColumn Header="Status">
    <tvs:TableViewTextColumn.CellStyle>
        <Style TargetType="tvs:TableViewCell">
            <Setter Property="AutomationProperties.Name"
                    Value="{Binding StatusDescription}" />
        </Style>
    </tvs:TableViewTextColumn.CellStyle>
</tvs:TableViewTextColumn>
```

### Custom Column Header Names

```xml
<tvs:TableViewTextColumn Header="DOB">
    <tvs:TableViewTextColumn.HeaderStyle>
        <Style TargetType="tvs:TableViewColumnHeader">
            <Setter Property="AutomationProperties.Name"
                    Value="Date of Birth" />
        </Style>
    </tvs:TableViewTextColumn.HeaderStyle>
</tvs:TableViewTextColumn>
```

### Custom Row Header Names

```xml
<tvs:TableView.RowHeaderTemplate>
    <DataTemplate>
        <!-- The TableViewRowHeader automation name derives from its content -->
        <TextBlock Text="{Binding EmployeeId}" />
    </DataTemplate>
</tvs:TableView.RowHeaderTemplate>
```

### Template Columns

For `TableViewTemplateColumn`, set `AutomationProperties.Name` directly on the template root element:

```xml
<tvs:TableViewTemplateColumn Header="Actions">
    <tvs:TableViewTemplateColumn.CellTemplate>
        <DataTemplate>
            <Button Content="Edit"
                    AutomationProperties.Name="Edit row" />
        </DataTemplate>
    </tvs:TableViewTemplateColumn.CellTemplate>
</tvs:TableViewTemplateColumn>
```

## Limitations

| Limitation | Notes |
|------------|-------|
| Virtualized cells | `IGridProvider.GetItem(row, column)` returns `null` for rows that are not currently realized in the visual tree. Scroll the target row into view first if needed. |
| Cell value via IValueProvider | `IValueProvider` is not implemented on `TableViewCellAutomationPeer`. Instead, the editing element (e.g. `TextBox`) exposes `IValueProvider` when the cell is in edit mode. |
| Non-Windows platforms | Automation peers are present on all platforms (WinUI 3 and Uno Platform). However, some automation clients and assistive technologies only run on Windows. |
| Custom column types | `TableViewTemplateColumn` cells display developer-defined content; accessible names for those cells default to the column header + row index unless overridden by the template content. |

## Uno Platform Notes

Automation peers compile and run on all Uno Platform targets. However, not all Uno Platform targets expose a full UIA tree to assistive technologies. On WebAssembly (WASM), the platform relies on ARIA attributes which Uno manages separately. On Desktop Skia targets, accessibility support depends on the platform's native accessibility APIs.

If a particular automation pattern is not needed for non-Windows targets, guard it with `#if WINDOWS` as shown in the rest of the codebase.
