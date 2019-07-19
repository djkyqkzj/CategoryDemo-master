# CategoryDemo-master
仿京东、拼多多商品分类页

![image](https://img-blog.csdn.net/2018041711082077?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxOTI3ODY1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



写这个需求的思路也很清晰，首先左边肯定是一个listView，右边也是一个listView，这两个listView要达到一个联动的效果。右边的listView再嵌套一个GridView即可。如下图所示。

![image](https://img-blog.csdn.net/2018041711101283?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxOTI3ODY1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2、将左侧数据和右侧数据分别渲染

2.1渲染左侧数据，即：

menuAdapter = new MenuAdapter(this, menuList);
lv_menu.setAdapter(menuAdapter);
1
2
很简单就不赘述了,底部会附上完整代码链接

2.2渲染右侧数据

 homeAdapter = new HomeAdapter(this, homeList);  lv_home.setAdapter(homeAdapter);

在HomeAdapter还需要在嵌套gridView，如下：

 @Override   public View getView(int position, View convertView, ViewGroup parent) {
    CategoryBean.DataBean dataBean = foodDatas.get(position);
    List<CategoryBean.DataBean.DataListBean> dataList = dataBean.getDataList();
    ViewHold viewHold = null;
    if (convertView == null) {
      convertView = View.inflate(context, R.layout.item_home, null);
      viewHold = new ViewHold();
      viewHold.gridView = (GridViewForScrollView) convertView.findViewById(R.id.gridView);
      viewHold.blank = (TextView) convertView.findViewById(R.id.blank);
      convertView.setTag(viewHold);
    } else {
      viewHold = (ViewHold) convertView.getTag();
    }
    HomeItemAdapter adapter = new HomeItemAdapter(context, dataList);
    viewHold.blank.setText(dataBean.getModuleTitle());
    viewHold.gridView.setAdapter(adapter);
    return convertView;   }

  private static class ViewHold {
    private GridViewForScrollView gridView;
    private TextView blank;   }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
这里需要说明两点，第一：由于listView嵌套gridView会有众所周知的问题，计算高度，所以这边需要重新测量高度，也就重写了gridView；第二：代码中的textView是每个子标题中主标题的名字，也就是需要吸在顶部的。

以上，就将数据已经可以渲染完成了，现在就是联动的问题

3、让两部分数据动起来

3.1 主数据联动子数据

只需要调用主数据的onItemClick()方法，右侧数据在复写方法中调用setSelection()方法即可

 lv_menu.setOnItemClickListener(new AdapterView.OnItemClickListener() {
      @Override
      public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
        menuAdapter.setSelectItem(position);
        menuAdapter.notifyDataSetInvalidated();
        tv_title.setText(menuList.get(position));
        lv_home.setSelection(showTitle.get(position));
      }
    });
1
2
3
4
5
6
7
8
9
3.2 子数据联动主数据

在onScroll中处理数据即可，在将主数据的adapter更新一下即可。如代码所示

lv_home.setOnScrollListener(new AbsListView.OnScrollListener() {
      private int scrollState;

      @Override
      public void onScrollStateChanged(AbsListView view, int scrollState) {
        this.scrollState = scrollState;
      }

      @Override
      public void onScroll(AbsListView view, int firstVisibleItem,
                 int visibleItemCount, int totalItemCount) {
        if (scrollState == AbsListView.OnScrollListener.SCROLL_STATE_IDLE) {
          return;
        }
        int current = showTitle.indexOf(firstVisibleItem);
        if (currentItem != current && current >= 0) {
          currentItem = current;
          tv_title.setText(menuList.get(currentItem));
          menuAdapter.setSelectItem(currentItem);
          menuAdapter.notifyDataSetInvalidated();
        }
      }
    });
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
4、吸顶效果

这边有几种方法都可以实现类似的效果，我这边是上面一直有一条，我再将子数据都加一个type，当type不同时，更换上面那一条的问题即可。
