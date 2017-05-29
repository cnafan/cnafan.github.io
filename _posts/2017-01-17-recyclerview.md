---
published: true
layout: post
title: RecyclerView多个布局
author: johnny 
category: articles
tags:
- recyclerview
- android
---
多布局是指一个不同的item用不同的layout显示。  
<!-- more -->
在xml中声明一个RecyclerView：
```
<android.support.v7.widget.RecyclerView
	android:layout_below="@id/toolbar"
    android:id="@+id/recyclerview"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
&lt;android.support.v7.widget.RecyclerView>
```
以及对应两种不同的item布局
```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    	android:id="@+id/country_withfirst"
    	android:layout_width="match_parent"
    	android:layout_height="wrap_content"
    	android:orientation="vertical">
	&lt;linearLayout
        	android:layout_width="match_parent"
        	android:layout_height="wrap_content"
        	android:background="@color/country_item_first_background">
        &lt;TextView
            	android:id="@+id/country_first"
            	android:layout_width="wrap_content"
            	android:layout_height="wrap_content"
            	android:layout_marginLeft="10dp"
            	android:gravity="left" />
    	&lt;/LinearLayout>
    &lt;/LinearLayout>
```
```
&lt;LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    	android:id="@+id/country_withoutfirst"
    	android:layout_width="match_parent"
    	android:layout_height="wrap_content"
    	android:background="@color/country_item_background"
    	android:orientation="horizontal"
    	android:paddingBottom="10dp"
    	android:paddingTop="10dp">
	&lt;TextView
        	android:id="@+id/country_item_normal_sum"
        	android:layout_width="wrap_content"
        	android:layout_height="wrap_content"
        	android:layout_marginLeft="10dp"
        	android:layout_weight="1"
        	android:gravity="left" />
    &lt;TextView
        	android:id="@+id/country_item_normal_count"
        	android:layout_width="wrap_content"
        	android:layout_height="wrap_content"
        	android:layout_marginRight="10dp"
        	android:layout_weight="1"
        	android:gravity="right"
        	android:textColor="@color/colorPrimary" />
	&lt;/LinearLayout>
```
mainActivity中
```
recyclerView = (RecyclerView) findViewById(R.id.recyclerview);
LinearLayoutManager layoutManager = new LinearLayoutManager(this);
recyclerView.setLayoutManager(layoutManager);
CountryAdapter adapter = new CountryAdapter(datas);
recyclerView.setAdapter(adapter);
```
实现自己的adapter
```
public class CountryAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
        List<CountryEntity> mDataList;

        CountryAdapter(List<CountryEntity> datas) {
            this.mDataList = datas;
            Log.d("CountryActivity", "datas.size=" + datas.size());
        }

        /**
         * 渲染具体的ViewHolder
         *
         * @param viewGroup ViewHolder的容器
         * @param i         一个标志，我们根据该标志可以实现渲染不同类型的ViewHolder
         * @return
         */
        @Override
        public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup viewGroup, int i) {
            if (i == NORMAL_ITEM) {
                return new NormalItemHolder(LayoutInflater.from(viewGroup.getContext()).inflate(R.layout.country_withoutfirst, viewGroup, false));
            } else {
                return new GroupItemHolder(LayoutInflater.from(viewGroup.getContext()).inflate(R.layout.country_withfirst, viewGroup, false));
            }
        }

        /**
         * 绑定ViewHolder的数据。
         *
         * @param holder
         * @param i      数据源list的下标
         */
        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder holder, int i) {
            if (holder instanceof NormalItemHolder) {
                NormalItemHolder h = (NormalItemHolder) holder;
                h.itemsum_normal.setText((datas.get(i)).getPublishDate());
                h.itemcount_normal.setText("+" + (datas.get(i)).getCountrycount());
            } else if (holder instanceof GroupItemHolder) {
                GroupItemHolder h = (GroupItemHolder) holder;
                h.itemfirst.setText((datas.get(i)).getItemFirst());
                h.itemsum_group.setText((datas.get(i)).getPublishDate());
                h.itemcount_group.setText("+" + (datas.get(i)).getCountrycount());
            }
        }

        @Override
        public int getItemCount() {
            return mDataList.size();
        }

        /**
         * 决定元素的布局使用哪种类型
         *
         * @param position 数据源List的下标
         * @return 一个int型标志，传递给onCreateViewHolder的第二个参数
         */
        @Override
        public int getItemViewType(int position) {
            //第一个都显示
            if (position == 0)
                return GROUP_ITEM;
            String currentDate = mDataList.get(position).getPublishDate();
            int prevIndex = position - 1;
            boolean isDifferent = !(checkFirst(mDataList.get(prevIndex).getPublishDate()) == (checkFirst(currentDate)));
            Log.d("CountryActivity","pre:"+mDataList.get(prevIndex).getPublishDate()+",now:"+currentDate);
            Log.d("CountryActivity","prefirst:"+(char)(checkFirst(mDataList.get(prevIndex).getPublishDate()))+",now:"+(char)(checkFirst(currentDate)));
            return isDifferent ? GROUP_ITEM : NORMAL_ITEM;
        }

        int checkFirst(String s) {
            return (int)(new ChineseCharToEn().getFirstLetter(s).charAt(0));
        }

        @Override
        public long getItemId(int position) {
            return mDataList.get(position).getItemId();
        }
```
对应两种不同的布局，实现两种不同的holder
```
/**
         * 标准
         */
        class NormalItemHolder extends RecyclerView.ViewHolder {
            TextView itemsum_normal;
            TextView itemcount_normal;

            NormalItemHolder(View itemView) {
                super(itemView);
                itemsum_normal = (TextView) itemView.findViewById(R.id.country_item_normal_sum);
                itemcount_normal = (TextView) itemView.findViewById(R.id.country_item_normal_count);
                itemView.findViewById(R.id.country_withoutfirst).setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        Intent backintent = new Intent();
                        Log.d("CountryActivity", "putstring:" + itemsum_normal.getText());
                        backintent.putExtra("name", itemsum_normal.getText());
                        backintent.putExtra("count", itemcount_normal.getText());
                        setResult(0, backintent);
                        finish();
                    }
                });
            }
        }
```
给每个item设置监听点击事件
```
/**
         * 带首字母
         */
        class GroupItemHolder extends RecyclerView.ViewHolder {
            TextView itemfirst;
            TextView itemsum_group;
            TextView itemcount_group;

            GroupItemHolder(View itemView) {
                super(itemView);
                itemfirst = (TextView) itemView.findViewById(R.id.country_first);
                itemsum_group = (TextView) itemView.findViewById(R.id.country_item_group_sum);
                itemcount_group = (TextView) itemView.findViewById(R.id.country_item_group_count);
                itemView.findViewById(R.id.country_withfirst).setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        Intent backintent = new Intent();
                        Log.d("CountryActivity", "putstring:" + itemsum_group.getText());
                        backintent.putExtra("name", itemsum_group.getText());
                        backintent.putExtra("count",itemcount_group.getText());
                        setResult(0, backintent);
                        finish();
                    }
                });
            }
        }
```
效果图如下：
![](/images/recyclerview_1.jpg)
