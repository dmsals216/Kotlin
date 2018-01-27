## Kotlin으로 만드는 RecyclerView

> 1. DBHelper 만들기, DB안에 데이터 넣기
> ```Kotlin
> class DBHelper(context: Context): SQLiteOpenHelper(context, "datadb", null, 1){
>    override fun onCreate(p0: SQLiteDatabase?) {
>        val tableSql = "create table tb_data (_id integer primary key autoincrement, name not null, date)"
>        p0!!.execSQL(tableSql)
>        p0!!.execSQL("insert into tb_data (name, date) values ('안영주', '2017-07-01')")
>        p0!!.execSQL("insert into tb_data (name, date) values ('최은경', '2017-07-01')")
>        p0!!.execSQL("insert into tb_data (name, date) values ('최호성', '2017-07-01')")
>        p0!!.execSQL("insert into tb_data (name, date) values ('정성택', '2017-06-30')")
>        p0!!.execSQL("insert into tb_data (name, date) values ('정길용', '2017-06-30')")
>        p0!!.execSQL("insert into tb_data (name, date) values ('임윤정', '2017-06-29')")
>        p0!!.execSQL("insert into tb_data (name, date) values ('김종덕', '2017-06-29')")
>        p0!!.execSQL("insert into tb_data (name, date) values ('채규태', '2017-06-28')")
>        p0!!.execSQL("insert into tb_data (name, date) values ('원형섭', '2017-06-28')")
>        p0!!.execSQL("insert into tb_data (name, date) values ('표선영', '2017-06-28')")
>    }
>    override fun onUpgrade(p0: SQLiteDatabase?, p1: Int, p2: Int) {
>        p0!!.execSQL("drop table tb_data")
>        onCreate(p0)
>    }
> }
> ```

> 2. Adapter안에 들어갈 list정의(onCreate에서 날짜 비교해서 데이터 넣기)
> ```Kotlin
> var list : MutableList<ItemV0>
>init {
>    list = arrayListOf()
>}
>
>override fun onCreate(savedInstanceState: Bundle?) {
>    super.onCreate(savedInstanceState)
>    setContentView(R.layout.activity_main)
>    val helper = DBHelper(this)
>    val db = helper.writableDatabase
>    val todayCursor = db.rawQuery("select * from tb_data where date = '2017-07-01'", null)
>    val yesterdayCursor = db.rawQuery("select * from tb_data where date = '2017-06-30'", null)
>    val before = db.rawQuery("select * from tb_data where date != '2017-07-01' and date != '2017-06-30'", null)
>    if(todayCursor.count != 0) {
>        val item = HeaderItem()
>        item.headerTitle = "오늘"
>        list.add(item)
>        while(todayCursor.moveToNext()) {
>            val dataItem = DataItem()
>            dataItem.name = todayCursor.getString(1)
>            dataItem.date = todayCursor.getString(2)
>            list.add(dataItem)
>        }
>    }
>
>    if(yesterdayCursor.count != 0) {
>        val item = HeaderItem()
>        item.headerTitle = "어제"
>        list.add(item)
>        while(yesterdayCursor.moveToNext()) {
>            val dataItem = DataItem()
>            dataItem.name = yesterdayCursor.getString(1)
>            dataItem.date = yesterdayCursor.getString(2)
>            list.add(dataItem)
>        }
>    }
>
>    if(before.count != 0) {
>        val item = HeaderItem()
>        item.headerTitle = "이전"
>        list.add(item)
>        while(before.moveToNext()) {
>            val dataItem = DataItem()
>            dataItem.name = before.getString(1)
>            dataItem.date = before.getString(2)
>            list.add(dataItem)
>        }
>    }
>
>    recyclerView.layoutManager = LinearLayoutManager(this)
>    recyclerView.adapter=MyAdapter(list)
>    recyclerView.addItemDecoration(MyDecoration())
> }

> 3. RecyclerView Adapter 정의
> ```Kotlin
> class MyAdapter(val list: MutableList<ItemV0>):RecyclerView.Adapter<RecyclerView.ViewHolder>() {
>    override fun getItemViewType(position: Int): Int {
>        return list.get(position).type
>    }
>
>    override fun getItemCount(): Int {
>        return list.size
>    }
>
>    override fun onBindViewHolder(holder: RecyclerView.ViewHolder?, position: Int) {
>        val itemV0 = list.get(position)
>        if(itemV0.type == ItemV0.TYPE_HEADER) {
>            val viewHolder = holder as HeaderViewHolder
>            val headerItem = itemV0 as HeaderItem
>            viewHolder.headerView.setText(headerItem.headerTitle)
>        }else {
>            val viewHolder = holder as DataViewHolder
>            val dataItem = itemV0 as DataItem
>            viewHolder.nameView.setText(dataItem.name)
>            viewHolder.dateView.setText(dataItem.date)
>
>            val count = position % 5
>            when(count) {
>                0 -> (viewHolder.personView.background as GradientDrawable).setColor(0xff009688.toInt())
>                1 -> (viewHolder.personView.background as GradientDrawable).setColor(0xff4285f4.toInt())
>                2 -> (viewHolder.personView.background as GradientDrawable).setColor(0xff039be5.toInt())
>                3 -> (viewHolder.personView.background as GradientDrawable).setColor(0xff9c27b0.toInt())
>                4 -> (viewHolder.personView.background as GradientDrawable).setColor(0xff0097a7.toInt())
>            }
>        }
>    }
>
>    override fun onCreateViewHolder(parent: ViewGroup?, viewType: Int): RecyclerView.ViewHolder {
>        if(viewType == ItemV0.TYPE_HEADER) {
>            val layoutInflater = LayoutInflater.from(parent?.context)
>            return HeaderViewHolder(layoutInflater.inflate(R.layout.item_header, parent, false))
>        }else {
>            val layoutInflater = LayoutInflater.from(parent?.context)
>            return DataViewHolder(layoutInflater.inflate(R.layout.item_data, parent, false))
>        }
>    }
> }
> ```

> 4. 데이터 정의
> ```Kotlin
> abstract class ItemV0 {
>    abstract val type : Int
>
>    companion object {
>        val TYPE_HEADER = 0
>        val TYPE_DATA = 1
>    }
>}
>
>class HeaderItem : ItemV0() {
>    var headerTitle: String? = null
>
>    override val type: Int
>        get() = ItemV0.TYPE_HEADER
>}
>
>internal class DataItem : ItemV0() {
>    var name: String? = null
>    var date: String? = null
>
>    override val type: Int
>        get() = ItemV0.TYPE_DATA
>}
> ```

> 5. Holder 정의
> ```Kotlin
> class HeaderViewHolder(view: View): RecyclerView.ViewHolder(view) {
>    val headerView = view.itemHeaderView
>}
>
>class DataViewHolder(view: View): RecyclerView.ViewHolder(view) {
>    val nameView = view.itemNameView
>    val dateView = view.itemDateView
>    val personView = view.itemPersonView
>}
> ```

> 6. 꾸미기(카드뷰를 이용x)
> ```Kotlin
> inner class MyDecoration:RecyclerView.ItemDecoration() {
>    override fun getItemOffsets(outRect: Rect?, view: View, parent: RecyclerView?, state: RecyclerView.State?) {
>        super.getItemOffsets(outRect, view, parent, state)
>        val index = parent!!.getChildAdapterPosition(view)
>        val itemV0 = list.get(index)
>        if(itemV0.type == ItemV0.TYPE_DATA) {
>            view!!.setBackgroundColor(0xFFFFFFFF.toInt())
>            ViewCompat.setElevation(view, 10.0f)
>        }
>        outRect!!.set(20, 10, 20, 10)
>    }
>}
> ```