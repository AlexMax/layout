Layout Reference
================

Functions
---------

```c++
void lay_init_context(lay_context *ctx);
```

Call this on a context before using it. You must also call this on a context if you would like to use it again after calling `lay_destroy_context()` on it.

```c++
void lay_reserve_items_capacity(lay_context *ctx, lay_id count);
```

Reserve enough heap memory to contain `count` items without needing to reallocate. The initial `lay_init_context()` call does not allocate any heap memory, so if you init a context and then call this once with a large enough number for the number of items you'll create, there will not be any further reallocations.

```c++
void lay_destroy_context(lay_context *ctx);
```

Frees any heap allocated memory used by a context. Don't call this on a context that did not have `lay_init_context()` call on it. To reuse a context after destroying it, you will need to call `lay_init_context()` on it again.

```c++
void lay_reset_context(lay_context *ctx);
```
Clears all of the items in a context, setting its count to 0. Use this when you want to re-declare your layout starting from the root item. This does not free any memory or perform allocations. It's safe to use the context again after calling this. You should probably use this instead of init/destroy if you are recalculating your layouts in a loop.

```c++
void lay_run_context(lay_context *ctx);
```

Performs the layout calculations, starting at the root item (id 0). After calling this, you can use lay_get_rect() to query for an item's calculated rectangle. If you use procedures such as `lay_append()` or `lay_insert()` after calling this, your calculated data may become invalid if a reallocation occurs.

You should prefer to recreate your items starting from the root instead of doing fine-grained updates to the existing context.

However, it's safe to use lay_set_size on an item, and then re-run lay_run_context. This might be useful if you are doing a resizing animation on items in a layout without any contents changing.

```c++
void lay_run_item(lay_context *ctx, lay_id item);
```

Like `lay_run_context()`, this procedure will run layout calculations -- however, it lets you specify which item you want to start from. `lay_run_context()` always starts with item 0, the first item, as the root. Running the layout calculations from a specific item is useful if you need to iteratively re-run parts of your layout hierarchy, or if you are only interested in updating certain subsets of it. Be careful when using this -- it's easy to generated bad output if the parent items haven't yet had their output rectangles calculated, or if they've been invalidated (e.g. due to re-allocation).

```c++
void lay_clear_item_break(lay_context *ctx, lay_id item);
```

Performing a layout on items where wrapping is enabled in the parent container can cause flags to be modified during the calculations. If you plan to call `lay_run_context` or `lay_run_item` multiple times without calling `lay_reset`, and if you have a container that uses wrapping, and if the width or height of the container may have changed, you should call `lay_clear_item_break` on all of the children of a container before calling `lay_run_context` or `lay_run_item` again. If you don't, the layout calculations may perform unnecessary wrapping. 

This requirement may be changed in the future.

Calling this will also reset any manually-specified breaking. You will need to set the manual breaking again, or simply not call this on any items that you know you wanted to break manually. 

If you clear your context every time you calculate your layout, or if you don't use wrapping, you don't need to call this.

```c++
lay_id lay_items_count(lay_context *ctx);
```

Returns the number of items that have been created in a context.

```c++
lay_id lay_items_capacity(lay_context *ctx);
```

Returns the number of items the context can hold without performing a reallocation.

```c++
lay_id lay_item(lay_context *ctx);
```

Create a new item, which can just be thought of as a rectangle. Returns the id (handle) used to identify the item.

```c++
void lay_insert(lay_context *ctx, lay_id parent, lay_id child);
```

Inserts an item into another item, forming a parent - child relationship. An item can contain any number of child items. Items inserted into a parent are put at the end of the ordering, after any existing siblings.

```c++
void lay_append(lay_context *ctx, lay_id earlier, lay_id later);
```

Inserts an item as a sibling after another item. This allows inserting an item into the middle of an existing list of items within a parent. It's also more efficient than repeatedly using `lay_insert(ctx, parent, new_child)` in a loop to create a list of items in a parent, because it does not need to traverse the parent's children each time. So if you're creating a long list of children inside of a parent, you might prefer to use this after using `lay_insert` to insert the first child.

```c++
void lay_push(lay_context *ctx, lay_id parent, lay_id child);
```

Like `lay_insert`, but puts the new item as the first child in a parent instead of as the last.

```c++
lay_vec2 lay_get_size(lay_context *ctx, lay_id item);
void lay_get_size_xy(lay_context *ctx, lay_id item, lay_scalar *x, lay_scalar *y);
```

Gets the size that was set with `lay_set_size` or `lay_set_size_xy`. The `_xy` version writes the output values to the specified addresses instead of returning the values in a `lay_vec2`.

```c++
void lay_set_size(lay_context *ctx, lay_id item, lay_vec2 size);
void lay_set_size_xy(lay_context *ctx, lay_id item, lay_scalar width, lay_scalar height);
```

Sets the size of an item. The `_xy` version passes the width and height as separate arguments, but functions the same.

```c++
void lay_set_contain(lay_context *ctx, lay_id item, uint32_t flags);
```

Set the flags on an item which determines how it behaves as a parent. For example, setting `LAY_COLUMN` will make an item behave as if it were a column -- it will lay out its children vertically.

```c++
void lay_set_behave(lay_context *ctx, lay_id item, uint32_t flags);
```

Set the flags on an item which determines how it behaves as a child inside of a parent item. For example, setting LAY_VFILL will make an item try to fill up all available vertical space inside of its parent.

```c++
lay_vec4 lay_get_margins(lay_context *ctx, lay_id item);
void lay_get_margins_ltrb(lay_context *ctx, lay_id item, lay_scalar *l, lay_scalar *t, lay_scalar *r, lay_scalar *b);
```

Get the margins that were set by `lay_set_margins`. The _ltrb version writes the output values to the specified addresses instead of returning the values in a lay_vec4. The `l`, `t`, `r` and `b` parameters stand for left, top, right and bottom respectively.

```c++
void lay_set_margins(lay_context *ctx, lay_id item, lay_vec4 ltrb);
```

Set the margins on an item. The four members of the vector represent, left, top, right, and bottom.

```c++
void lay_set_margins_ltrb(lay_context *ctx, lay_id item, lay_scalar l, lay_scalar t, lay_scalar r, lay_scalar b);
```

Same as lay_set_margins, but the components are passed as separate arguments.

Contain Flags
-------------

Bitfield flags you can pass to `lay_set_container`.

Flex direction:

- **LAY_ROW**: Left to right
- **LAY_COLUMN**: Top to bottom

Model:

- **LAY_LAYOUT**: Free layout
- **LAY_FLEX**: Flex model

Flex wrap:

- **LAY_NOWRAP**: Single-line
- **LAY_WRAP**: Multi-line, wrap left to right

Justify content:

- **LAY_START**: At start of row/column
- **LAY_MIDDLE**: At center of row/column
- **LAY_END**: At end of row/column
- **LAY_JUSTIFY**: Insert spacing to stretch across whole row/column

Child Layout (Behavior) Flags
-----------------------------

Bitfield flags you can pass to `lay_set_behave`.

Attachments - fully valid when parent uses `LAY_LAYOUT` model, partially valid when in `LAY_FLEX` model.

- **LAY_LEFT**: Anchor to left item or left side of parent.
- **LAY_TOP**: Anchor to top item or top side of parent.
- **LAY_RIGHT**: Anchor to right item or right side of parent.
- **LAY_BOTTOM**: Anchor to bottom item or bottom side of parent.
- **LAY_HFILL**: Anchor to both left and right item or parent borders.
- **LAY_VFILL**: Anchor to both top and bottom item or parent borders.
- **LAY_HCENTER**: Center horizontally, with left margin as offset.
- **LAY_VCENTER**: Center vertically, with top margin as offset.
- **LAY_CENTER**: Center in both directions, with left/top margin as offset.
- **LAY_FILL**: Anchor to all four directions.
- **LAY_BREAK**
  When in a wrapping container, put this element on a new line. Wrapping layout code auto-inserts `LAY_BREAK` flags as needed. See GitHub issues for TODO related to this.
  
  Drawing routines can read this via item pointers as needed after performing layout calculations.
