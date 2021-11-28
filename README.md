学院：计算机与网络空间安全  
专业：软件工程  
学号：116052019023  
姓名：林嘉豪  
项目名称：基于NotePad应用的功能扩展  
——————————————————————————————————————————  

一、实验要求  
        阅读NotePad的源代码并做如下扩展：
        基本要求：
        NoteList中显示条目增加时间戳显示
        添加笔记查询功能（根据标题查询）
   扩展要求：  
        附加功能：根据自身实际情况进行扩充（至少两项）
        
二、实验内容  
- 基本功能  
1. NoteList中显示条目增加时间戳显示   
[image](https://github.com/qwwqq/test1/blob/master/app/images/1.png)

2.添加笔记查询功能（根据标题查询）  
所有笔记：  
[image](https://github.com/qwwqq/test1/blob/master/app/images/1.png)

搜索带ljh关键字的便签：  
[image](https://github.com/qwwqq/test1/blob/master/app/images/2.png)

- 扩展功能  
1.美化UI界面：  
[image](https://github.com/qwwqq/test1/blob/master/app/images/3.png)
[image](https://github.com/qwwqq/test1/blob/master/app/images/5.png)


2.导出笔记：  
[image](https://github.com/qwwqq/test1/blob/master/app/images/4.png)

三、关键代码  
1.NoteList中显示条目增加时间戳显示  
1）实现时间戳的布局框架：  
 <!--添加显示时间的TextView-->  
    <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"/>

2）在NoteList类的PROJECTION中添加时间字段  

        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE

3）增加一个ID给显示时间用  

        int[] viewIDs = { android.R.id.text1 ,android.R.id.text2}

4）在note editor获取并格式化时间：  

        ContentValues values = new ContentValues();
        Long now = Long.valueOf(System.currentTimeMillis());
        SimpleDateFormat sf = new SimpleDateFormat("yy/MM/dd HH:mm");
        Date d = new Date(now);
        String format = sf.format(d);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, format);

2.添加笔记查询功能（根据标题查询）  
search功能具体实现代码：

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        SearchView searchView = findViewById(R.id.search_view);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listView = findViewById(R.id.list_view);
        sqLiteDatabase = new NotePadProvider.DatabaseHelper(this).getReadableDatabase();
        //设置该SearchView显示搜索按钮
        searchView.setSubmitButtonEnabled(true);
        //设置该SearchView内默认显示的提示文本
        searchView.setQueryHint("查找");
        searchView.setOnQueryTextListener(this);
    }
    public boolean onQueryTextChange(String string) {
        String selection1 = NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?";
        String[] selection2 = {"%"+string+"%","%"+string+"%"};
        Cursor cursor = sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION, // The columns to return from the query
                selection1, // The columns for the where clause
                selection2, // The values for the where clause
                null,          // don't group the rows
                null,          // don't filter by row groups
                NotePad.Notes.DEFAULT_SORT_ORDER // The sort order
        );
        // The names of the cursor columns to display in the view, initialized to the title column
        String[] dataColumns = {
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
        } ;
        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = {
                android.R.id.text1,
                android.R.id.text2
        };
        // Creates the backing adapter for the ListView.
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,         // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );
        // Sets the ListView's adapter to be the cursor adapter that was just created.
        listView.setAdapter(adapter);
        return true;
    }
}

3.美化UI  
1）五种颜色的类替换：  

public static final int DEFAULT_COLOR = 0; //白  
public static final int YELLOW_COLOR = 1; //黄  
public static final int BLUE_COLOR = 2; //蓝  
public static final int GREEN_COLOR = 3; //绿  
public static final int RED_COLOR = 4; //红  

2）颜色填充的具体实现： 

public class MyCursorAdapter extends SimpleCursorAdapter {  
    public MyCursorAdapter(Context context, int layout, Cursor c,  
                           String[] from, int[] to) {  
        super(context, layout, c, from, to);  
    }  
    @Override  
    public void bindView(View view, Context context, Cursor cursor){  
        super.bindView(view, context, cursor);  
        //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色  
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));  
        /**  
         * 白 255 255 255  
         * 黄 247 216 133  
         * 蓝 165 202 237  
         * 绿 161 214 174  
         * 红 244 149 133  
         */  
        switch (x){  
            case NotePad.Notes.DEFAULT_COLOR:  
                view.setBackgroundColor(Color.rgb(255, 255, 255));  
                break;  
            case NotePad.Notes.YELLOW_COLOR:  
                view.setBackgroundColor(Color.rgb(247, 216, 133));  
                break;  
            case NotePad.Notes.BLUE_COLOR:  
                view.setBackgroundColor(Color.rgb(165, 202, 237));  
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;  
            case NotePad.Notes.RED_COLOR:  
                view.setBackgroundColor(Color.rgb(244, 149, 133));   
                break;  
            default:  
                view.setBackgroundColor(Color.rgb(255, 255, 255));  
                break;  
        }  
    }  
}  

3）在NoteList类的PROJECTION中显示颜色字段：

        NotePad.Notes.COLUMN_NAME_BACK_COLOR
        
4.笔记的导出  
1）导出界面布局：  

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
    <EditText android:id="@+id/output_name"
        android:maxLines="1"
        android:layout_marginTop="2dp"
        android:layout_marginBottom="15dp"
        android:layout_width="wrap_content"
        android:ems="25"
        android:layout_height="wrap_content"
        android:autoText="true"
        android:capitalize="sentences"
        android:scrollHorizontally="true" />
    <Button android:id="@+id/output_ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:text="@string/output_ok"
        android:onClick="OutputOk" />
</LinearLayout>

2）便签导出方法具体实现： 
···
public class OutputText extends Activity {  
   //要使用的数据库中笔记的信息  
    private static final String[] PROJECTION = new String[] {  
            NotePad.Notes._ID, // 0  
            NotePad.Notes.COLUMN_NAME_TITLE, // 1  
            NotePad.Notes.COLUMN_NAME_NOTE, // 2  
            NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 3  
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 4  
    };  
    //读取出的值放入这些变量  
    private String TITLE;  
    private String NOTE;  
    private String CREATE_DATE;  
    private String MODIFICATION_DATE;  
    //读取该笔记信息  
    private Cursor mCursor;  
    //导出文件的名字  
    private EditText mName;  
    //NoteEditor传入的uri，用于从数据库查出该笔记  
    private Uri mUri;  
    //关于返回与保存按钮的一个特殊标记，返回的话不执行导出，点击按钮才导出  
    private boolean flag = false;  
    private static final int COLUMN_INDEX_TITLE = 1;  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.output_text);  
        mUri = getIntent().getData();  
        mCursor = managedQuery(  
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
        mName = (EditText) findViewById(R.id.output_name);
    }
    @Override
    protected void onResume(){
        super.onResume();
        if (mCursor != null) {
            // The Cursor was just retrieved, so its index is set to one record *before* the first
            // record retrieved. This moves it to the first record.
            mCursor.moveToFirst();
            //编辑框默认的文件名为标题，可自行更改
            mName.setText(mCursor.getString(COLUMN_INDEX_TITLE));
        }
    }
    @Override
    protected void onPause() {
        super.onPause();
        if (mCursor != null) {
        //从mCursor读取对应值
            TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
            NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
            CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
            MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
            //flag在点击导出按钮时会设置为true，执行写文件
            if (flag == true) {
                write();
            }
            flag = false;
        }
    }
    public void OutputOk(View v){
        flag = true;
        finish();
    }
    private void write()
    {
        try
        {
            // 如果手机插入了SD卡，而且应用程序具有访问SD的权限
            if (Environment.getExternalStorageState().equals(
                    Environment.MEDIA_MOUNTED)) {
                // 获取SD卡的目录
                File sdCardDir = Environment.getExternalStorageDirectory();
                //创建文件目录
                File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
                //写文件
                PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                ps.println(TITLE);
                ps.println(NOTE);
                ps.println("创建时间：" + CREATE_DATE);
                ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                ps.close();
                Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
···
