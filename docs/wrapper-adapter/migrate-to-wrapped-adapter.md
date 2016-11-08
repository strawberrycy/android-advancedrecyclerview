# Tweak your Adapter to support adapter wrapping

Need to tweak several things in your adapter implemtation to support wrapping adapter mechanism.
Because `WrapperAdapter` modifies **item positions**, **item IDs** and **item view types** internally.

[TOC]

## If using ViewHolder.getAdaperPosition() / ViewHolder.getLayoutPosition())

👉 &nbsp; Use `WrapperAdapterUtils.unwrapPosition()`.

```java
@Overfides
void onClick(View v) {
    RecyclerView rv = RecyclerViewAdapterUtils.getParentRecyclerView(v);
    RecyclerView.ViewHolder vh = rv.findContainingViewHolder(v);

    int rootPosition = vh.getAdapterPosition();
    if (rootPosition == RecyclerView.NO_POSITION) {
        return;
    }

    // need to determine adapter local position like this:
    RecyclerView.Adapter rootAdapter = rv.getAdapter();
    int localPosition = WrapperAdapterUtils.unwrapPosition(rootAdapter, this, rootPosition);

    Item item = mItems.get(localPosition);
    ...
}
```

## If using ViewHolder.getItemViewType()

👉 &nbsp; Use `ItemViewTypeComposer.extractWrappedViewTypePart()`.

```java
@Overfides
void onClick(View v) {
    RecyclerView.ViewHolder vh = recyclerView.findContainingViewHolder(v);

    int rawViewType = vh.getItemViewType();
    int viewType = ItemIdComposer.extractWrappedIdPart(rawViewType);

    // use "viewType" here to determine which type of item is clicked
    ...
}
```


## If using ViewHolder.getItemId()

👉 &nbsp; Use `ItemIdComposer.extractWrappedIdPart()`.

```java
@Overfides
void onClick(View v) {
    RecyclerView.ViewHolder vh = recyclerView.findContainingViewHolder(v);

    long rawId = vh.getItemId();
    long id = ItemIdComposer.extractWrappedIdPart(rawId);

    // use "id" here to determine which item is clicked
    ...
}
```


## If your adapter overrides optional methods of RecyclerView.Adapter

👉 &nbsp; Implement the `WrappedAdapter` interface and use the `viewType` parameter instead of using `ViewHolder.getItemViewType()`

This step is only required when you are overriding the following methods;

- `onViewAttachedToWindow()`
- `onViewDetachedFromWindow()`
- `onViewRecycled()`
- `onFailedToRecycleView()`

```java
class MyInnerAdapter<VH> implements WrappedAdapter<VH> {
    // the following four methods are provided by WrappedAdapter interface
    @Overrides
    void onViewAttachedToWindow(VH holder, int viewType) { ... }

    @Overrides
    void onViewDetachedFromWindow(VH holder, int viewType) { ... }

    @Overrides
    void onViewRecycled(VH holder, int viewType) { ... }

    @Overrides
    boolean onFailedToRecycleView(VH holder, int viewType) { ... }

    // proxy to WrappedAdapter's methods
    @Overrides
    void onViewAttachedToWindow(VH holder) {
        onViewAttachedToWindow(holder, holder.getItemViewType());
    }

    @Overrides
    void onViewDetachedFromWindow(VH holder, int viewType) {
        onViewDetachedFromWindow(holder, holder.getItemViewType());
    }

    @Overrides
    void onViewRecycled(VH holder, int viewType) {
        onViewRecycled(holder, holder.getItemViewType());
    }

    @Overrides
    boolean onFailedToRecycleView(VH holder, int viewType) {
        onFailedToRecycleView(holder, holder.getItemViewType());
    }
}
```