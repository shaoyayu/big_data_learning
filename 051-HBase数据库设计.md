# HBase数据库设计

## 题目：

1、人员-角色
  人员有多个角色  角色优先级
  角色有多个人员
  人员 删除添加角色
  角色 可以添加删除人员
  人员 角色 删除添加

2、组织架构 部门-子部门
  查询 顶级部门
  查询 每个部门的所有子部门
  部门 添加、删除子部门
  部门 添加、删除 

3、通话记录：
手机号		对方手机号		通话时长		时间		通话类型（主叫/被叫）

## 通话记录设计



```java
package com.bjsxt.hbase;

......

public class PhoneCase {

	// 表的管理类
	HBaseAdmin admin = null;
	// 数据的管理类
	HTable table = null;
	// 表名
	String tm = "phone";

	/**
	 * 完成初始化功能
	 * 
	 * @throws Exception
	 */
	@Before
	public void init() throws Exception {
		Configuration conf = new Configuration();
		conf.set("hbase.zookeeper.quorum", "node1,node2,node3");
		admin = new HBaseAdmin(conf);
		table = new HTable(conf, tm.getBytes());
	}

	/**
	 * 创建表
	 * 
	 * @throws Exception
	 */
	@Test
	public void createTable() throws Exception {
		// 表的描述类
		HTableDescriptor desc = new HTableDescriptor(TableName.valueOf(tm));
		// 列族的描述类
		HColumnDescriptor family = new HColumnDescriptor("cf".getBytes());
		desc.addFamily(family);
		if (admin.tableExists(tm)) {
			admin.disableTable(tm);
			admin.deleteTable(tm);
		}
		admin.createTable(desc);
	}

	/**
	 * 10个用户，每个用户每年产生1000条通话记录
	 * 
	 * dnum:对方手机号 type:类型：0主叫，1被叫 length：长度 date:时间
	 * 
	 * @throws Exception
	 * 
	 */
	@Test
	public void insert() throws Exception {
		List<Put> puts = new ArrayList<Put>();
		for (int i = 0; i < 10; i++) {
			String phoneNumber = getPhone("158");
			for (int j = 0; j < 1000; j++) {
				// 属性
				String dnum = getPhone("177");
				String length = String.valueOf(r.nextInt(99));
				String type = String.valueOf(r.nextInt(2));
				String date = getDate("2018");
				// rowkey设计
				String rowkey = phoneNumber + "_" + (Long.MAX_VALUE - sdf.parse(date).getTime());
				Put put = new Put(rowkey.getBytes());
				put.add("cf".getBytes(), "dnum".getBytes(), dnum.getBytes());
				put.add("cf".getBytes(), "length".getBytes(), length.getBytes());
				put.add("cf".getBytes(), "type".getBytes(), type.getBytes());
				put.add("cf".getBytes(), "date".getBytes(), date.getBytes());
				puts.add(put);
			}
		}
		table.put(puts);
	}

	/**
	 * 查询某一个用户3月份的所有通话记录 条件： 1、某一个用户 2、时间
	 * 
	 * @throws Exception
	 */
	@Test
	public void scan() throws Exception {
		String phoneNumber = "15895223166";
		String startRow = phoneNumber + "_" + (Long.MAX_VALUE - sdf.parse("20180401000000").getTime());
		String stopRow = phoneNumber + "_" + (Long.MAX_VALUE - sdf.parse("20180301000000").getTime());

		Scan scan = new Scan();
		scan.setStartRow(startRow.getBytes());
		scan.setStopRow(stopRow.getBytes());
		ResultScanner scanner = table.getScanner(scan);
		for (Result result : scanner) {
			System.out.print(Bytes
					.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "dnum".getBytes()))));
			System.out.print("--" + Bytes
					.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "type".getBytes()))));
			System.out.print("--" + Bytes
					.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "date".getBytes()))));
			System.out.println("--" + Bytes
					.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "length".getBytes()))));
		}
	}

	/**
	 * 查询某一个用户。所有的主叫电话 条件： 1、电话号码 2、type=0
	 * 
	 * @throws Exception
	 */
	@Test
	public void scan2() throws Exception {
		FilterList filters = new FilterList(FilterList.Operator.MUST_PASS_ALL);
		SingleColumnValueFilter filter1 = new SingleColumnValueFilter("cf".getBytes(), "type".getBytes(),
				CompareOp.EQUAL, "0".getBytes());
		PrefixFilter filter2 = new PrefixFilter("15895223166".getBytes());
		filters.addFilter(filter1);
		filters.addFilter(filter2);

		Scan scan = new Scan();
		scan.setFilter(filters);
		ResultScanner scanner = table.getScanner(scan);
		for (Result result : scanner) {
			System.out.print(Bytes
					.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "dnum".getBytes()))));
			System.out.print("--" + Bytes
					.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "type".getBytes()))));
			System.out.print("--" + Bytes
					.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "date".getBytes()))));
			System.out.println("--" + Bytes
					.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "length".getBytes()))));
		}
		scanner.close();
	}

	/**
	 * 
	 * @param string
	 * @return
	 */

	private String getDate(String string) {
		return string + String.format("%02d%02d%02d%02d%02d", r.nextInt(12) + 1, r.nextInt(31), r.nextInt(24),
				r.nextInt(60), r.nextInt(60));
	}

	Random r = new Random();
	SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddhhmmss");

	private String getPhone(String phonePrefix) {
		return phonePrefix + String.format("%08d", r.nextInt(99999999));
	}

	/**
	 * 10个用户，每个用户1000条，每一条记录当作一个对象进行存储
	 * @throws Exception 
	 */
	@Test
	public void insert2() throws Exception{
		List<Put> puts = new ArrayList<Put>();
		for(int i = 0;i<10;i++){
			String phoneNumber = getPhone("158");
			for(int j = 0;j<1000;j++){
				String dnum = getPhone("177");
				String length =String.valueOf(r.nextInt(99));
				String type = String.valueOf(r.nextInt(2));
				String date = getDate("2018");
				//保存属性到对象中
				Phone.PhoneDetail.Builder phoneDetail = Phone.PhoneDetail.newBuilder();
				phoneDetail.setDate(date);
				phoneDetail.setLength(length);
				phoneDetail.setType(type);
				phoneDetail.setDnum(dnum);
				
				//rowkey
				String rowkey = phoneNumber+"_"+(Long.MAX_VALUE-sdf.parse(date).getTime());
			
				Put put = new Put(rowkey.getBytes());
				put.add("cf".getBytes(), "phoneDetail".getBytes(), phoneDetail.build().toByteArray());
				puts.add(put);
			}
		}
		table.put(puts);
	}
	
	@Test
	public void get() throws Exception{
		Get get = new Get("15866626435_9223370522183722807".getBytes());
		Result result = table.get(get);
		PhoneDetail phoneDetail = Phone.PhoneDetail.parseFrom(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "phoneDetail".getBytes())));
		System.out.println(phoneDetail);
	}
	
	/**
	 * 10个用户，每天产生1000条记录，每一天的所有数据放到一个rowkey中
	 * @throws Exception 
	 */
	@Test
	public void insert3() throws Exception{
		List<Put> puts = new ArrayList<Put>();

		for(int i = 0;i<10;i++){
			String phoneNumber = getPhone("133");
			String rowkey = phoneNumber+"_"+(Long.MAX_VALUE-sdf.parse("20181225000000").getTime());
			Phone.DayOfPhone.Builder dayOfPhone = Phone.DayOfPhone.newBuilder();
			for(int j=0;j<1000;j++){
				String dnum = getPhone("177");
				String length =String.valueOf(r.nextInt(99));
				String type = String.valueOf(r.nextInt(2));
				String date = getDate2("20181225");
				
				Phone.PhoneDetail.Builder phoneDetail = Phone.PhoneDetail.newBuilder();
				phoneDetail.setDate(date);
				phoneDetail.setLength(length);
				phoneDetail.setType(type);
				phoneDetail.setDnum(dnum);
				dayOfPhone.addDayPhone(phoneDetail);
			}
			Put put = new Put(rowkey.getBytes());
			put.add("cf".getBytes(), "day".getBytes(), dayOfPhone.build().toByteArray());
			puts.add(put);
		}
		table.put(puts);
		
	}
	
	@Test
	public void get2() throws Exception{
		Get get = new Get("13398049199_9223370491187575807".getBytes());
		Result result = table.get(get);
		DayOfPhone parseFrom = Phone.DayOfPhone.parseFrom(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "day".getBytes())));
		int count = 0;
		for (PhoneDetail pd : parseFrom.getDayPhoneList()) {
			System.out.println(pd);
			count++;
		}
		System.out.println(count);
	}
	
	private String getDate2(String string) {
		return string+String.format("%02d%02d%02d", r.nextInt(24),r.nextInt(60),r.nextInt(60));
	}

	@After
	public void destory() throws Exception {
		
		if (admin != null) {
			admin.close();
		}
	}
}

```



## 人员-角色

```shell
两张表
psn
rowkey：					cf1:(人员信息)						cf2:(角色列表)
pid
001						cf1:name=小红						cf2:100=10,cf2:200=9
002						cf1:name=小白						cf2:200=10,cf2:300=9
003						cf1:name=小黑						cf2:300=10,cf2:400=9
004						cf1:name=小绿				

role
rowkey:						cf1：(角色信息)						cf2:(人员列表)
rid	
100						cf1:name=班长						cf2:001=小红
200						cf1:name=学委						cf2:001=小红，cf2:002=小白
300						cf1:name=劳委						cf2:001=小黑，cf2:002=小白
400						cf1:name=体委						cf2:001=小黑，

```

### 协处理器

到目前为止，用户已经掌握了如何使用过滤器来减少服务器端通过网络返回到客户端的数据量。HBase 中还有一些特性让用户 甚至可以把一部分计算也移动到数据的存放端:协处理器( coprocessor)。

## 组织架构

```shell
CEO300	
	develop	100
		develop1	200
		develop2	400
	web800
		web1 700
		web2 1100
	test	600
		test1	500
		test2	900
		test3	1000

dept
rowkey:						cf:(部门信息)					cf2:(子部门列表)
did
300							cf:name=ceo,					cf2:100=,cf2:800=?,cf2:600=?
100							cf:name=develop,cf:fid=300		cf2:200=,
200							cf:name=develop1,cf:fid=100
400							cf:name=develop2,cf:fid=100
800							cf:name=web,cf:fid=300			cf2:700=?,cf:1100=?
700							cf:name=web1,cf:fid=800					
1100 						cf:name=web2,cf:fid=800
```

