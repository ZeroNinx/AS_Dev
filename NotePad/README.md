# Notepad - 安卓期中作业
### 总览

基本要求实现：增删改查，时间戳，笔记查询（查询所有文本关键字）

拓展要求实现：UI美化，笔记背景更改，笔记分类及查询

## 详情

### SQlite数据库表格实现

<table>
  <tr>
    <th>列名</th>
    <th>类型</th>
    <th>描述</th>
  </tr>
  <tr>
    <td>id</td>
    <td>int primary key</td>
    <td>主键，标识文本号</td>
  </tr>
  <tr>
    <td>note_title</td>
    <td>text</td>
    <td>标题</td>
  </tr>
  <tr>
    <td>note_text</td>
    <td>text</td>
    <td>内容</td>
  </tr>
  <tr>
    <td>note_tag</td>
    <td>text</td>
    <td>分类</td>
  </tr>
  <tr>
    <td>note_time</td>
    <td>datetime</td>
    <td>时间戳</td>
  </tr>
  <tr>
    <td>background_color</td>
    <td>int</td>
    <td>背景色</td>
  </tr>
</table>
 
操作API(自封装：)
```Java

//主页
public class MainActivity extends AppCompatActivity
{
    //SQLite数据库
    protected SQLiteDatabase db;

    //载入数据库
    public void loadDB()
    {
        db = SQLiteDatabase.openOrCreateDatabase(getFilesDir().toString() + "/notepad.db", null);
    }

    //关闭数据库
    public void closeDB()
    {
        db.close();
    }

    //生成表
    {
        String create_table = "create table if not exists notepad(" +
                              "id integer primary key," +
                              "note_title text," +
                              "note_text text," +
                              "note_tag text default '默认'," +
                              "note_time datetime," +
                              "background_color integer)";
        db.execSQL(create_table);
    }
}
```
  
### 主界面

<img src='https://github.com/ZeroNinx/AS_Dev/blob/master/NotePad/screenshot/main.png' width='350px' />

主要是参考安卓5.0以后的风格做了界面美化，由于安卓5.0后更新的扁平化Api更加美观，同时加上系统的支持，一体性更加强。

主要代码（style.xml）：

```xml
    <resources>
        <!-- Base application theme. -->
        <style name="AppTheme" parent="Theme.AppCompat.DayNight.DarkActionBar">
            <!-- Customize your theme here. -->
            <item name="colorPrimary">#208ADF</item>
            <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
            <item name="colorAccent">@color/colorAccent</item>
            <item name="android:windowTranslucentStatus">true</item>
        </style>
    </resources>
```

菜单包括分类筛选，搜索，以及多选。其中分类筛选和搜索功能用MenuItem的回调函数实现，多选使用ActionMode实现，主要列表是用了SimpleAdaptor装配器实现。

关键代码：

```Java
//主页
public class MainActivity extends AppCompatActivity
{
    //初始化列表
    protected void initList(String keyword)
    {
        //......
        //构造组件映射列表
        ArrayList<HashMap<String, Object>> list = new ArrayList<HashMap<String, Object>>();

        //查找并装配
        String sql = "select * from notepad;";

        //重载关键字
        if (keyword != null)
            sql = "select * from notepad where note_title like '%" + keyword + "%'" +
                  " or note_text like '%" + keyword + "%'" +
                  " or note_tag like '%" + keyword + "%';";

        Cursor result = db.rawQuery(sql, null);
        while (result.moveToNext())
        {
            //获取参数列
            int idColumn = result.getColumnIndex("id");
            int titleColumn = result.getColumnIndex("note_title");
            int textColumn = result.getColumnIndex("note_text");
            //.....

            //设置映射
            HashMap<String, Object> mp = new HashMap<String, Object>();
            mp.put("tv_id", result.getInt(idColumn));
            mp.put("iv_icon", R.drawable.icon_notepad);
            //.....

            list.add(mp);
        }

        result.close();
        closeDB();

        //设定装配器
        SimpleAdapter sa = new SimpleAdapter(this, list, R.layout.lv_index_unit,
                new String[]{"tv_id", "iv_icon", "tv_title", "tv_text", "tv_time", "tv_tag"},
                new int[]{R.id.tv_id, R.id.iv_icon, R.id.tv_title, R.id.tv_text, R.id.tv_time, R.id.tv_tag});

        //装配
        lv_index.setAdapter(sa);
        //......
    }
}
```

考虑到用户的体验，为了避免关键字漏查的情况，搜索功能对标题、文本、分类等涉及的字段都进行查找。多选操作可以长按，迎合传统习惯。

图标和文字也统一使用了黑白配的风格，加强一体性。

代码：MainActivity

<img src='https://github.com/ZeroNinx/AS_Dev/blob/master/NotePad/screenshot/multiselect.png' width='250px' /><img src='https://github.com/ZeroNinx/AS_Dev/blob/master/NotePad/screenshot/seatch.png' width='250px' /><img src='https://github.com/ZeroNinx/AS_Dev/blob/master/NotePad/screenshot/select_tag.png' width='250px' />

### 笔记界面

笔记界面以易用性为目的，主要是用字体区分标题栏，统一位置的分类筛选，同时在菜单上利用Layout实现了自定义的图标显示以及点击操作。

美化工作集中在文本编辑背景下划线（代码适配）以及更改背景颜色两个方面。

背景颜色更改使用了SeekBar提及其回调函数实现，利用了单独实现的Activity而不是弹出框加强定制化。

笔记编辑设置了更加美观的行高和间距，下划线功能有安卓原生bug（以解决），这个Bug似乎在高版本中解决了。

代码：ActivityNotepad 

调色板：PaletteActivity

自定义文本框View：MultilineTextEditWithUnderLine

<img src='https://github.com/ZeroNinx/AS_Dev/blob/master/NotePad/screenshot/notepad.png' width='250px' /><img src='https://github.com/ZeroNinx/AS_Dev/blob/master/NotePad/screenshot/palette.png' width='250px' /><img src='https://github.com/ZeroNinx/AS_Dev/blob/master/NotePad/screenshot/set_tag.png' width='250px' />
