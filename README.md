# 基于NotePad应用的功能扩展
 ## 基本功能实现
 ### （1）增加时间戳显示
  ![图片alt](https://github.com/aishuqing/NotePad/blob/master/app/时间戳.PNG)
  
  在xml布局文件里对其添加一个黑色的textview
  ```
  <!--添加一个垂直的显示布局-->
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"
        android:singleLine="true"
        />

    <!--添加 显示时间 的TextView-->
    <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"/>

</LinearLayout>
  
 
  ```
  ```
  
  通过Cursor从数据库中读取出：

Cursor cursor = managedQuery(
            getIntent().getData(),            // Use the default content URI for the provider.
            PROJECTION,                       // Return the note ID and title for each note.
            null,                             // No where clause, return all records.
            null,                             // No where clause, therefore no where column values.
            NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
通过SimpleCursorAdapter装填：

String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE } ;
int[] viewIDs = { android.R.id.text1 };
SimpleCursorAdapter adapter
    = new SimpleCursorAdapter(
            this, // The Context for the ListView
            R.layout.noteslist_item, // Points to the XML for a list item
            cursor,   // The cursor to get items from
            dataColumns,
            viewIDs
    );
// Sets the ListView's adapter to be the cursor adapter that was just created.
setListAdapter(adapter);
  ```
  
 ### （2）根据标题查找
  ![图片alt](https://github.com/aishuqing/NotePad/blob/master/app/search1.PNG)
  ![图片alt](https://github.com/aishuqing/NotePad/blob/master/app/search2.PNG)
  
  ```
  笔记查询
找到菜单的xml文件，list_options_menu.xml，添加一个搜索的item，搜索图标用安卓自带的图标，设为总是显示：

<item
    android:id="@+id/menu_search"
    android:title="@string/menu_search"
    android:icon="@android:drawable/ic_search_category_default"
    android:showAsAction="always">
</item>
在NotesList中找到onOptionsItemSelected方法，在switch中添加搜索的case语句:

 //添加搜素
    case R.id.menu_search:
    Intent intent = new Intent();
    intent.setClass(NotesList.this,NoteSearch.class);
    NotesList.this.startActivity(intent);
    return true;

  ```
  ```
  。先布局搜索页面，在layout中新建布局文件note_search_list.xml：

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="输入搜索内容..."
        android:layout_alignParentTop="true">
    </SearchView>
    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
  ```
## 拓展功能解析
### 更改背景
 #### （1）原本图片
 ![图片alt](https://github.com/aishuqing/NotePad/blob/master/app/更改记事本的背景1.PNG) 
 
 #### （2）
 ![图片alt](https://github.com/aishuqing/NotePad/blob/master/app/更改背景2.PNG)
 
 #### （3）更改后的图片背景 
 ![图片alt](https://github.com/aishuqing/NotePad/blob/master/app/背景3.PNG) 
 
 
 ```
 背景更换主要是让NotesList和NoteSearch每条笔记都有背景色，并且能保存。要做到保存颜色的数据，最直接的办法就是在数据库中添加一个颜色的字段，在这之前在NotePad契约类中添加：

public static final String COLUMN_NAME_BACK_COLOR = "color";
 ```
```
创建数据库表地方添加颜色的字段：

 @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + "   ("
        + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
        + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
        + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
        + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
        + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
        + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //颜色
        + ");");
       }
把颜色定义为INTEGER的主要原因是，在系统中预定于好五种颜色，根据颜色对应不int值选择要显示的颜色，契约类中的定义：

public static final int DEFAULT_COLOR = 0; //白
public static final int YELLOW_COLOR = 1; //黄
public static final int BLUE_COLOR = 2; //蓝
public static final int GREEN_COLOR = 3; //绿
public static final int RED_COLOR = 4; //红
```
```
NotesList中的PROJECTION添加颜色项：

private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
并且将NotesList中用的SimpleCursorAdapter改使用MyCursorAdapter：

 //修改为可以填充颜色的自定义的adapter，自定义的代码在MyCursorAdapter.java中
adapter = new MyCursorAdapter(
        this,
        R.layout.noteslist_item,
        cursor,
        dataColumns,
        viewIDs
    );
```
