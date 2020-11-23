---
title: "Implement Endless Scroll with RecyclerView"
layout: post
excerpt: ""
categories: [Android]
tags: [Java, Android]
---
## Implementation Idea

1. Distinguish different items
    you can define base Item class like this
    ```java
    public abstract class ListItem {
        public final Type mType;
        protected ListItem(Type type){
            mType = type;
        }
        public int typeToInteger(){
            return mType.toInteger();
        }
        enum Type{
            NORMAL(0), PROGRESS(1);
            private int mOrder;

            Type(int type){
                mOrder = type;
            }

            public int toInteger(){
                return mOrder;
            }
        }
    }
    ```

2. Write RecyclerView Adapter
    ```java
    public class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyViewHolder>{
        private final List<ListItem> mDataSet;
        private final RecyclerView mRecyclerView;

        public MyAdapter(List<ListItem> dataSet, RecyclerView recyclerView){
            mDataSet = dataSet;
            mRecyclerView = recyclerView;
        }

        @Override
        public int getItemViewType(int position) {
            ListItem item = mDataSet.get(position);
            return item.mType.toInteger();
        }

        @Override
        public int getItemCount() {
            return mDataSet.size();
        }

        public static class MyViewHolder extends RecyclerView.ViewHolder{
            public final ListItem.Type mType;
            public MyViewHolder(View v, ListItem.Type type){
                super(v);
                mType = type;
            }
        }
    }
    ```
    the ```ListItem.Type``` will be used useful to distinguish between them.

3. Now Inheritance ```class MyViewHolder``` and ```class ListItem```

    - Items
        ```java
        public class ProgressItem extends ListItem{
            public ProgressItem(){
                super(Type.PROGRESS);
            }
        }
        ```
        ```java
        public class NormalItem extends ListItem{
            private String text;
            public NormalItem(String data){
                super(Type.NORMAL);
                text = data;
            }
            public String getText(){
                return text;
            }
        }
        ```
    - ViewHolders    
        The ```MyAdapter``` can use these ```ViewHolder``` for the ```onCreateViewHolder```.
        ```java
        public class ProgressViewHolder extends MyAdapter.MyViewHolder{
            public ProgressViewHolder(View v){
                super(v, ListItem.Type.PROGRESS);
            }
        }
        ```
        ```java
        public class NormalViewHolder extends MyAdapter.MyViewHolder{
            public final TextView mTextView;

            public NormalViewHolder(View v) {
                super(v, ListItem.Type.NORMAL);
                mTextView = (TextView) v.findViewById(R.id.txt_field);
            }
        }
        ```
4. Override RecyclerView.Adapter's methods (```class MyAdapter```)
    ```java
    @NonNull
    @Override
    public MyViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View v;
        if(viewType == ListItem.Type.NORMAL.toInteger()){
            v = LayoutInflater.from(parent.getContext())
                    .inflate(R.layout.recycler_item, parent, false);
            return new NormalViewHolder(v);
        }
        else{//progress
            v = LayoutInflater.from(parent.getContext())
                    .inflate(R.layout.progress, parent, false);
            return new ProgressViewHolder(v);
        }
    }

    @Override
    public void onBindViewHolder(@NonNull MyViewHolder holder, int position) {
        if(holder.mType == ListItem.Type.NORMAL){
            NormalItem tmp = (NormalItem) mDataSet.get(position);
            ((NormalViewHolder) holder).mTextView.setText(tmp.getText());
        }
    }

    @Override
    public int getItemViewType(int position) {
        ListItem item = mDataSet.get(position);
        return item.mType.toInteger();
    }

    @Override
    public int getItemCount() {
        return mDataSet.size();
    }
    ```
    Now, you can understand why the ```ListItem.Type``` is needed.
    __```recyclerView.Adapter``` provides the ```getItemViewType```__ so that we can distinguish between the ListItems.
    When RecyclerView creates a new View for ListItem, the Adapter can load different layouts depends on ```ListItem.type```
    In this example, I used __if-else__ cause there are only two different kinds of ListItem.
5. Finally, you can use it!    
    for display progress bar
    ```java
    private void setProgress(){
        mAdapter.addItem(new ProgressItem());
        mAdapter.notifyDataSetChanged();
    }
    ```
    you can add ```RecyclerView.OnScrollListener```    
    [reference_stack_over_flow](https://stackoverflow.com/questions/36127734/detect-when-recyclerview-reaches-the-bottom-most-position-while-scrolling)
    ```java
    recyclerView.addOnScrollListener(new RecyclerView.OnScrollListener() {
        @Override
        public void onScrollStateChanged(@NonNull RecyclerView recyclerView, int newState) {
            super.onScrollStateChanged(recyclerView, newState);
            if(!recyclerView.canScrollVertically(1) && newState == RecyclerView.SCROLL_STATE_IDLE){
                doTask();
            }
        }
    });
    ```
    the ```doTask()``` is ```AsyncTask``` for load more data from the source.
