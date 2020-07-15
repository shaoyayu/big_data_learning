# HBase API 



## 说明

这里我配置的是HBase-0.98.23-hadoop2，api也是使用的是这个版本的



### pom文件

```xml
<!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-client/0.98.23-hadoop2 -->
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>0.98.23-hadoop2</version>
</dependency>
```

### 官网案例

```java
package icu.shaoyayu.hadoop.hbase;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HConstants;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.io.compress.Compression.Algorithm;
/**
 * @author shaoyayu
 * @date 2020/7/10 8:35
 * @E_Mail
 * @Version 1.0.0
 * @readme ：
 */
public class Example {
    private static final String TABLE_NAME = "MY_TABLE_NAME_TOO";
    private static final String CF_DEFAULT = "DEFAULT_COLUMN_FAMILY";

    public static void createOrOverwrite(Admin admin, HTableDescriptor table) throws IOException {
        if (admin.tableExists(table.getTableName())) {
            admin.disableTable(table.getTableName());
            admin.deleteTable(table.getTableName());
        }
        admin.createTable(table);
    }

    public static void createSchemaTables(Configuration config) throws IOException {
        try (Connection connection = ConnectionFactory.createConnection(config);
             Admin admin = connection.getAdmin()) {

            HTableDescriptor table = new HTableDescriptor(TableName.valueOf(TABLE_NAME));
            table.addFamily(new HColumnDescriptor(CF_DEFAULT).setCompressionType(Algorithm.NONE));

            System.out.print("Creating table. ");
            createOrOverwrite(admin, table);
            System.out.println(" Done.");
        }
    }

    public static void modifySchema (Configuration config) throws IOException {
        try (Connection connection = ConnectionFactory.createConnection(config);
             Admin admin = connection.getAdmin()) {

            TableName tableName = TableName.valueOf(TABLE_NAME);
            if (!admin.tableExists(tableName)) {
                System.out.println("Table does not exist.");
                System.exit(-1);
            }

            HTableDescriptor table = admin.getTableDescriptor(tableName);

            // Update existing table
            HColumnDescriptor newColumn = new HColumnDescriptor("NEWCF");
            newColumn.setCompactionCompressionType(Algorithm.GZ);
            newColumn.setMaxVersions(HConstants.ALL_VERSIONS);
            admin.addColumn(tableName, newColumn);

            // Update existing column family
            HColumnDescriptor existingColumn = new HColumnDescriptor(CF_DEFAULT);
            existingColumn.setCompactionCompressionType(Algorithm.GZ);
            existingColumn.setMaxVersions(HConstants.ALL_VERSIONS);
            table.modifyFamily(existingColumn);
            admin.modifyTable(tableName, table);

            // Disable an existing table
            admin.disableTable(tableName);

            // Delete an existing column family
            admin.deleteColumn(tableName, CF_DEFAULT.getBytes("UTF-8"));

            // Delete a table (Need to be disabled first)
            admin.deleteTable(tableName);
        }
    }

    public static void main(String... args) throws IOException {
        Configuration config = HBaseConfiguration.create();

        //Add any necessary configuration files (hbase-site.xml, core-site.xml)
        config.addResource(new Path(System.getenv("HBASE_CONF_DIR"), "hbase-site.xml"));
        config.addResource(new Path(System.getenv("HADOOP_CONF_DIR"), "core-site.xml"));
        createSchemaTables(config);
        modifySchema(config);
    }
}

```



### 代码测试1

```java
package icu.shaoyayu.hadoop.hbase;

import static org.junit.Assert.assertTrue;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.io.InterruptedIOException;
import java.util.List;

/**
 * @author shaoyayu
 */
public class AppTest {
    //表的管理类
    HBaseAdmin admin = null;

    //表数据管理类
    HTable hTable = null;

    //表名
    String tm = "phone";
    //rowkey
    String rowKey = "rk1";

    //列族
    String family = "fm";

    /**
     * 初始化环境
     */
    @Before
    public void initializeTheEnvironment() throws IOException {
        Configuration configuration = new CompoundConfiguration();
        //连接Zookeeper配置
        configuration.set("hbase.zookeeper.quorum","hadoopNode02,hadoopNode03,hadoopNode04");
        admin = new HBaseAdmin(configuration);
        hTable = new HTable(configuration,tm.getBytes());
    }

    /**
     * 创建一张表
     * @throws IOException
     */
    @Test
    public void createTable() throws IOException {
        //创建表名
        TableName tableName = TableName.valueOf(tm);
        HTableDescriptor ht = new HTableDescriptor(tableName);
        //添加列族
        HColumnDescriptor fam = new HColumnDescriptor(family.getBytes());
        ht.addFamily(fam);
        //判断是否已经存在
        if (admin.tableExists(tm)){
            //清除表数据
            admin.disableTable(tm);
            //删除表
            admin.deleteTable(tm);
        }
        admin.createTable(ht);

    }

    /**
     * 插入数据
     * @throws InterruptedIOException
     * @throws RetriesExhaustedWithDetailsException
     */
    @Test
    public void insertData() throws InterruptedIOException, RetriesExhaustedWithDetailsException {
        //构造参数需要一个rowKey
        Put put = new Put(rowKey.getBytes());
        put.add(family.getBytes(),"name".getBytes(),"zhangsan".getBytes());
        put.add(family.getBytes(),"age".getBytes(),"22".getBytes());
        put.add(family.getBytes(),"sex".getBytes(),"man".getBytes());
        hTable.put(put);
    }

    /**
     * 查询数据
     * @throws IOException
     */
    @Test
    public void queryData() throws IOException {
        Get get = new Get(rowKey.getBytes());
        //添加一个在服务器上的过滤操作，减少网络i/o
        get.addColumn(family.getBytes(),"name".getBytes());
        Result result = hTable.get(get);
        Cell cell = result.getColumnLatestCell(family.getBytes(), "name".getBytes());
        byte[] bytes = CellUtil.cloneValue(cell);
        String name = Bytes.toString(bytes);
        System.out.println(name);
    }

    /**
     * 查询整张表
     * @throws IOException
     */
    @Test
    public void queryTheEntireTable() throws IOException {
        Scan scan = new Scan();
        //可以通过rowkey来限定Scan查询传来的数据量
//        scan.setStartRow();
//        scan.setStopRow();
        ResultScanner scanner = hTable.getScanner(scan);
        for (Result result : scanner) {
            Cell cell = result.getColumnLatestCell(family.getBytes(), "name".getBytes());
            byte[] bytes = CellUtil.cloneValue(cell);
            String name = Bytes.toString(bytes);
            System.out.println(name);
        }
    }

    @After
    public void destory(){
        if (admin!=null){
            try {
                admin.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```





### 代码测试2

```java
package icu.shaoyayu.hadoop.hbase;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.KeyValue;
import org.apache.hadoop.hbase.MasterNotRunningException;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.ZooKeeperConnectionException;
import org.apache.hadoop.hbase.client.Delete;
import org.apache.hadoop.hbase.client.Get;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.client.HConnection;
import org.apache.hadoop.hbase.client.HConnectionManager;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.HTableInterface;
import org.apache.hadoop.hbase.client.HTablePool;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.filter.BinaryComparator;
import org.apache.hadoop.hbase.filter.CompareFilter.CompareOp;
import org.apache.hadoop.hbase.filter.Filter;
import org.apache.hadoop.hbase.filter.FilterList;
import org.apache.hadoop.hbase.filter.PrefixFilter;
import org.apache.hadoop.hbase.filter.RowFilter;
import org.apache.hadoop.hbase.filter.SingleColumnValueFilter;
import org.apache.hadoop.hbase.filter.SubstringComparator;
import org.apache.hadoop.hbase.util.Bytes;
import org.junit.Test;


public class HBaseDAOImp {

	HConnection hTablePool = null;
	static Configuration conf =null;
	public HBaseDAOImp()
	{
		 conf = new Configuration();
		String zk_list = "node1,node2,node3";
		conf.set("hbase.zookeeper.quorum", zk_list);
		try {
			hTablePool = HConnectionManager.createConnection(conf) ;
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	public void save(Put put, String tableName) {
		// TODO Auto-generated method stub
		HTableInterface table = null;
		try {
			table = hTablePool.getTable(tableName) ;
			table.put(put) ;
			
		} catch (Exception e) {
			e.printStackTrace() ;
		}finally{
			try {
				table.close() ;
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
	
	/**
	 * 插入一个cell
	 * @param tableName
	 * @param rowKey
	 * @param family
	 * @param quailifer
	 * @param value
	 */
	public void insert(String tableName, String rowKey, String family,
			String quailifer, String value) {
		// TODO Auto-generated method stub
		HTableInterface table = null;
		try {
			table = hTablePool.getTable(tableName) ;
			Put put = new Put(rowKey.getBytes());
			put.add(family.getBytes(), quailifer.getBytes(), value.getBytes()) ;
			table.put(put);
		} catch (Exception e) {
			e.printStackTrace();
		}finally
		{
			try {
				table.close() ;
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
	
	/**
	 * 在一个列族下插入多个单元格
	 * @param tableName
	 * @param rowKey
	 * @param family
	 * @param quailifer
	 * @param value
	 */
	public void insert(String tableName,String rowKey,String family,String quailifer[],String value[])
	{
		HTableInterface table = null;
		try {
			table = hTablePool.getTable(tableName) ;
			Put put = new Put(rowKey.getBytes());
			// 批量添加
			for (int i = 0; i < quailifer.length; i++) {
				String col = quailifer[i];
				String val = value[i];
				put.add(family.getBytes(), col.getBytes(), val.getBytes());
			}
			table.put(put);
		} catch (Exception e) {
			e.printStackTrace();
		}finally
		{
			try {
				table.close() ;
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
	public void save(List<Put> Put, String tableName) {
		// TODO Auto-generated method stub
		HTableInterface table = null;
		try {
			table = hTablePool.getTable(tableName) ;
			table.put(Put) ;
		}
		catch (Exception e) {
			// TODO: handle exception
		}finally
		{
			try {
				table.close() ;
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		
	}


	public Result getOneRow(String tableName, String rowKey) {
		// TODO Auto-generated method stub
		HTableInterface table = null;
		Result rsResult = null;
		try {
			table = hTablePool.getTable(tableName) ;
			Get get = new Get(rowKey.getBytes()) ;
			rsResult = table.get(get) ;
		} catch (Exception e) {
			e.printStackTrace() ;
		}
		finally
		{
			try {
				table.close() ;
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return rsResult;
	}
	
	/**
	 * 最常用的方法，优化查询
	 * 查询一行数据，
	 * @param tableName
	 * @param rowKey
	 * @param cols
	 * @return
	 */
	public Result getOneRowAndMultiColumn(String tableName, String rowKey,String[] cols) {
		// TODO Auto-generated method stub
		HTableInterface table = null;
		Result rsResult = null;
		try {
			table = hTablePool.getTable(tableName) ;
			Get get = new Get(rowKey.getBytes()) ;
			for (int i = 0; i < cols.length; i++) {
				get.addColumn("cf".getBytes(), cols[i].getBytes()) ;
			}
			rsResult = table.get(get) ;
		} catch (Exception e) {
			e.printStackTrace() ;
		}
		finally
		{
			try {
				table.close() ;
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return rsResult;
	}

	
	public List<Result> getRows(String tableName, String rowKeyLike) {
		// TODO Auto-generated method stub
		HTableInterface table = null;
		List<Result> list = null;
		try {
			FilterList fl = new FilterList(FilterList.Operator.MUST_PASS_ALL);
			table = hTablePool.getTable(tableName) ;
			PrefixFilter filter = new PrefixFilter(rowKeyLike.getBytes());
			SingleColumnValueFilter filter1 = new SingleColumnValueFilter(
					  "order".getBytes(),
					  "order_type".getBytes(),
					  CompareOp.EQUAL,
					  Bytes.toBytes("1")
					  );
			fl.addFilter(filter);
			fl.addFilter(filter1);
			Scan scan = new Scan();
			scan.setFilter(fl);
			ResultScanner scanner = table.getScanner(scan) ;
			list = new ArrayList<Result>() ;
			for (Result rs : scanner) {
				list.add(rs) ;
			}
		} catch (Exception e) {
			e.printStackTrace() ;
		}
		finally
		{
			try {
				table.close() ;
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return list;
	}
	
	
	public List<Result> getRows(String tableName, String rowKeyLike ,String cols[]) {
		// TODO Auto-generated method stub
		HTableInterface table = null;
		List<Result> list = null;
		try {
			table = hTablePool.getTable(tableName) ;
			PrefixFilter filter = new PrefixFilter(rowKeyLike.getBytes());
			
			Scan scan = new Scan();
			for (int i = 0; i < cols.length; i++) {
				scan.addColumn("cf".getBytes(), cols[i].getBytes()) ;
			}
			scan.setFilter(filter);
			ResultScanner scanner = table.getScanner(scan) ;
			list = new ArrayList<Result>() ;
			for (Result rs : scanner) {
				list.add(rs) ;
			}
		} catch (Exception e) {
			e.printStackTrace() ;
		}
		finally
		{
			try {
				table.close() ;
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return list;
	}
	
	public List<Result> getRowsByOneKey(String tableName, String rowKeyLike ,String cols[]) {
		// TODO Auto-generated method stub
		HTableInterface table = null;
		List<Result> list = null;
		try {
			table = hTablePool.getTable(tableName) ;
			PrefixFilter filter = new PrefixFilter(rowKeyLike.getBytes());
			
			Scan scan = new Scan();
			for (int i = 0; i < cols.length; i++) {
				scan.addColumn("cf".getBytes(), cols[i].getBytes()) ;
			}
			scan.setFilter(filter);
			ResultScanner scanner = table.getScanner(scan) ;
			list = new ArrayList<Result>() ;
			for (Result rs : scanner) {
				list.add(rs) ;
			}
		} catch (Exception e) {
			e.printStackTrace() ;
		}
		finally
		{
			try {
				table.close() ;
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return list;
	}
	
	/**
	 * 范围查询
	 * @param tableName
	 * @param startRow
	 * @param stopRow
	 * @return
	 */
	public List<Result> getRows(String tableName,String startRow,String stopRow)
	{
		HTableInterface table = null;
		List<Result> list = null;
		try {
			table = hTablePool.getTable(tableName) ;
			Scan scan = new Scan() ;
			scan.setStartRow(startRow.getBytes()) ;
			scan.setStopRow(stopRow.getBytes()) ;
			ResultScanner scanner = table.getScanner(scan) ;
			list = new ArrayList<Result>() ;
			for (Result rsResult : scanner) {
				list.add(rsResult) ;
			}
			
		}catch (Exception e) {
			e.printStackTrace() ;
		}
		finally
		{
			try {
				table.close() ;
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return list;
	}
	
	
	public void deleteRecords(String tableName, String rowKeyLike){
		HTableInterface table = null;
		try {
			table = hTablePool.getTable(tableName) ;
			PrefixFilter filter = new PrefixFilter(rowKeyLike.getBytes());
			Scan scan = new Scan();
			scan.setFilter(filter);
			ResultScanner scanner = table.getScanner(scan) ;
			List<Delete> list = new ArrayList<Delete>() ;
			for (Result rs : scanner) {
				Delete del = new Delete(rs.getRow());
				list.add(del) ;
			}
			table.delete(list);
		}
		catch (Exception e) {
			e.printStackTrace() ;
		}
		finally
		{
			try {
				table.close() ;
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		
	}
	
	public void deleteCell(String tableName, String rowkey,String cf,String column){
		HTableInterface table = null;
		try {
			table = hTablePool.getTable(tableName) ;
			Delete del = new Delete(rowkey.getBytes());
			del.deleteColumn(cf.getBytes(), column.getBytes());
			table.delete(del);
		}
		catch (Exception e) {
			e.printStackTrace() ;
		}
		finally
		{
			try {
				table.close() ;
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		
	}
	
	public   void createTable(String tableName, String[] columnFamilys){
		try {
			// admin 对象
			HBaseAdmin admin = new HBaseAdmin(conf);
			if (admin.tableExists(tableName)) {
				System.err.println("此表，已存在！");
			} else {
				HTableDescriptor tableDesc = new HTableDescriptor(
						TableName.valueOf(tableName));

				for (String columnFamily : columnFamilys) {
					tableDesc.addFamily(new HColumnDescriptor(columnFamily));
				}

				admin.createTable(tableDesc);
				System.err.println("建表成功!");

			}
			admin.close();// 关闭释放资源
		} catch (MasterNotRunningException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (ZooKeeperConnectionException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}

	/**
	 * 删除一个表
	 * 
	 * @param tableName
	 *            删除的表名
	 * */
	public   void deleteTable(String tableName)   {
		try {
			HBaseAdmin admin = new HBaseAdmin(conf);
			if (admin.tableExists(tableName)) {
				admin.disableTable(tableName);// 禁用表
				admin.deleteTable(tableName);// 删除表
				System.err.println("删除表成功!");
			} else {
				System.err.println("删除的表不存在！");
			}
			admin.close();
		} catch (MasterNotRunningException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (ZooKeeperConnectionException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
 
	/**
	 * 查询表中所有行
	 * @param tablename
	*/
	public void scaner(String tablename) {
	try {
	        HTable table =new HTable(conf, tablename);
	        Scan s =new Scan();
//	        s.addColumn(family, qualifier)
//	        s.addColumn(family, qualifier)
	        ResultScanner rs = table.getScanner(s);
	for (Result r : rs) {
	            
		   for(Cell cell:r.rawCells()){   
			    System.out.println("RowName:"+new String(CellUtil.cloneRow(cell))+" ");
			    System.out.println("Timetamp:"+cell.getTimestamp()+" ");
			    System.out.println("column Family:"+new String(CellUtil.cloneFamily(cell))+" ");
			    System.out.println("row Name:"+new String(CellUtil.cloneQualifier(cell))+" ");
			    System.out.println("value:"+new String(CellUtil.cloneValue(cell))+" ");
			   }
	        }
	    } catch (IOException e) {
	        e.printStackTrace();
	    }
	}
	public void scanerByColumn(String tablename) {
		
		try {
			HTable table =new HTable(conf, tablename);
			Scan s =new Scan();
	        s.addColumn("cf".getBytes(), "201504052237".getBytes());
	        s.addColumn("cf".getBytes(), "201504052237".getBytes());
			ResultScanner rs = table.getScanner(s);
			for (Result r : rs) {
				
				for(Cell cell:r.rawCells()){   
					System.out.println("RowName:"+new String(CellUtil.cloneRow(cell))+" ");
					System.out.println("Timetamp:"+cell.getTimestamp()+" ");
					System.out.println("column Family:"+new String(CellUtil.cloneFamily(cell))+" ");
					System.out.println("row Name:"+new String(CellUtil.cloneQualifier(cell))+" ");
					System.out.println("value:"+new String(CellUtil.cloneValue(cell))+" ");
				}
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	public static void main(String[] args) {
		

		
		
//		创建表
//	    String tableName="test";
//		String cfs[] = {"cf"};
//		dao.createTable(tableName,cfs);
		
//		存入一条数据
//		Put put = new Put("bjsxt".getBytes());
//		put.add("cf".getBytes(), "name".getBytes(), "cai10".getBytes()) ;
//		dao.save(put, "test") ;

//		插入多列数据
//		Put put = new Put("bjsxt".getBytes());
//		List<Put> list = new ArrayList<Put>();
//		put.add("cf".getBytes(), "addr".getBytes(), "shanghai1".getBytes()) ;
//		put.add("cf".getBytes(), "age".getBytes(), "30".getBytes()) ;
//		put.add("cf".getBytes(), "tel".getBytes(), "13889891818".getBytes()) ;
//		list.add(put) ;
//		dao.save(list, "test");
		
//		插入单行数据
//		dao.insert("test", "testrow", "cf", "age", "35") ;
//		dao.insert("test", "testrow", "cf", "cardid", "12312312335") ;
//		dao.insert("test", "testrow", "cf", "tel", "13512312345") ;
		
		
		
//		List<Result> list = dao.getRows("test", "testrow",new String[]{"age"}) ;
//		for(Result rs : list)
//		{
//		    for(Cell cell:rs.rawCells()){   
//		        System.out.println("RowName:"+new String(CellUtil.cloneRow(cell))+" ");
//		        System.out.println("Timetamp:"+cell.getTimestamp()+" ");
//		        System.out.println("column Family:"+new String(CellUtil.cloneFamily(cell))+" ");
//		        System.out.println("row Name:"+new String(CellUtil.cloneQualifier(cell))+" ");
//		        System.out.println("value:"+new String(CellUtil.cloneValue(cell))+" ");
//		       }
//		}
		
//		Result rs = dao.getOneRow("test", "testrow");
//		 System.out.println(new String(rs.getValue("cf".getBytes(), "age".getBytes())));
		
//		Result rs = dao.getOneRowAndMultiColumn("cell_monitor_table", "29448-513332015-04-05", new String[]{"201504052236","201504052237"});
//		for(Cell cell:rs.rawCells()){   
//	        System.out.println("RowName:"+new String(CellUtil.cloneRow(cell))+" ");
//	        System.out.println("Timetamp:"+cell.getTimestamp()+" ");
//	        System.out.println("column Family:"+new String(CellUtil.cloneFamily(cell))+" ");
//	        System.out.println("row Name:"+new String(CellUtil.cloneQualifier(cell))+" ");
//	        System.out.println("value:"+new String(CellUtil.cloneValue(cell))+" ");
//	       }
		
//		dao.deleteTable("cell_monitor_table");
//		创建表
	    String tableName="cell_monitor_table";
		String cfs[] = {"cf"};
//		dao.createTable(tableName,cfs);
	}
	
	
	  public static void testRowFilter(String tableName){
	     	try {
	     		 HTable table =new HTable(conf, tableName);
				Scan scan = new Scan();
				scan.addColumn(Bytes.toBytes("column1"),Bytes.toBytes("qqqq"));
				Filter filter1 = new RowFilter(CompareOp.LESS_OR_EQUAL,new BinaryComparator(Bytes.toBytes("laoxia157")));
				scan.setFilter(filter1);
				ResultScanner scanner1 = table.getScanner(scan);
				for (Result res : scanner1) {
				    System.out.println(res);
				}
				scanner1.close();

//			 
//				Filter filter2 = new RowFilter(CompareFilter.CompareOp.EQUAL,new RegexStringComparator("laoxia4\\d{2}"));
//				scan.setFilter(filter2);
//				ResultScanner scanner2 = table.getScanner(scan);
//				for (Result res : scanner2) {
//				     System.out.println(res);
//				}
//				scanner2.close();

				Filter filter3 = new RowFilter( CompareOp.EQUAL,new SubstringComparator("laoxia407"));
				scan.setFilter(filter3);
				ResultScanner scanner3 = table.getScanner(scan);
				for (Result res : scanner3) {
				      System.out.println(res);
				}
				scanner3.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
	    }
	    
	  @Test
	  public void testTrasaction(){
		  try{
			  HTableInterface table = null;
			  table = hTablePool.getTable("t_test".getBytes());
//			  Put put1 =new Put("002".getBytes());
//			  put1.add("cf1".getBytes(), "name".getBytes(), "王五".getBytes());
//			  table.put(put1);
			  Put newput =new Put("001".getBytes());
			  newput.add("cf1".getBytes(), "like".getBytes(), "看书".getBytes());
			  
			  boolean f= table.checkAndPut("001".getBytes(), "cf1".getBytes(), "age".getBytes(), "24".getBytes(), newput);
			  System.out.println(f);
			  
		  }catch (Exception e){
			  e.printStackTrace();
		  }
		  
	  }
}

```



### [过滤器](https://hbase.apache.org/book.html#client.filter)

#### FilterList

```java
FilterList list = new FilterList(FilterList.Operator.MUST_PASS_ONE);
SingleColumnValueFilter filter1 = new SingleColumnValueFilter(
  cf,
  column,
  CompareOperator.EQUAL,
  Bytes.toBytes("my value")
  );
list.add(filter1);
SingleColumnValueFilter filter2 = new SingleColumnValueFilter(
  cf,
  column,
  CompareOperator.EQUAL,
  Bytes.toBytes("my other value")
  );
list.add(filter2);
scan.setFilter(list);
```

####  [SingleColumnValueFilter](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/filter/SingleColumnValueFilter.html)

can be used to test column values for equivalence (`CompareOperaor.EQUAL`), inequality (`CompareOperaor.NOT_EQUAL`), or ranges (e.g., `CompareOperaor.GREATER`). The following is an example of testing equivalence of a column to a String value "my value"…

```java
SingleColumnValueFilter filter = new SingleColumnValueFilter(
  cf,
  column,
  CompareOperaor.EQUAL,
  Bytes.toBytes("my value")
  );
scan.setFilter(filter);
```

### Individual Filter Syntax

#### KeyOnlyFilter

This filter doesn’t take any arguments. It returns only the key component of each key-value.

此过滤器不接受任何参数。 它仅返回每个键值的键组成部分。

#### FirstKeyOnlyFilter

This filter doesn’t take any arguments. It returns only the first key-value from each row.

此过滤器不接受任何参数。 它仅返回每行的第一个键值。

#### PrefixFilter

This filter takes one argument – a prefix of a row key. It returns only those key-values present in a
row that starts with the specified row prefix

此过滤器采用一个参数–行键的前缀。 它仅返回包含在
以指定的行前缀开头的行

#### ColumnPrefixFilter

This filter takes one argument – a column prefix. It returns only those key-values present in a
column that starts with the specified column prefix. The column prefix must be of the form:
“qualifier”.

该过滤器采用一个参数–列前缀。 它仅返回包含在
以指定列前缀开头的列。 列前缀的格式必须为：
“预选qualifier”

#### MultipleColumnPrefixFilter

This filter takes a list of column prefixes. It returns key-values that are present in a column that
starts with any of the specified column prefixes. Each of the column prefixes must be of the form:
“qualifier”.

该过滤器采用列前缀列表。 它返回存在于以下列中的键值：
以任何指定的列前缀开头。 每个列前缀都必须采用以下格式：
“qualifier”。

#### ColumnCountGetFilter

This filter takes one argument – a limit. It returns the first limit number of columns in the table.

此过滤器接受一个参数-一个限制。 它返回表中的第一个限制列数。

#### PageFilter

This filter takes one argument – a page size. It returns page size number of rows from the table.

此过滤器采用一个参数–页面大小。 它从表中返回页面大小的行数。

#### ColumnPaginationFilter

This filter takes two arguments – a limit and offset. It returns limit number of columns after offset
number of columns. It does this for all the rows.

该过滤器采用两个参数–限制和偏移量。 偏移量后返回限制的列数
列数。 它对所有行都执行此操作。

#### InclusiveStopFilter

This filter takes one argument – a row key on which to stop scanning. It returns all key-values
present in rows up to and including the specified row.

该过滤器采用一个参数–行键，停止扫描。 它返回所有键值
出现在指定行之前（包括指定行）。

#### TimeStampsFilter

This filter takes a list of timestamps. It returns those key-values whose timestamps matches any of
the specified timestamps.

该过滤器采用时间戳列表。 它返回那些时间戳与以下任何一个匹配的键值
指定的时间戳。

#### RowFilter

This filter takes a compare operator and a comparator. It compares each row key with the
comparator using the compare operator and if the comparison returns true, it returns all the keyvalues in that row.

该滤波器带有一个比较运算符和一个比较器。 它将每个行键与
比较器使用compare运算符，如果比较返回true，则返回该行中的所有键值。

#### Family Filter

This filter takes a compare operator and a comparator. It compares each column family name with
the comparator using the compare operator and if the comparison returns true, it returns all the
Cells in that column family.

该滤波器带有一个比较运算符和一个比较器。 它将每个列的姓氏与
比较器使用compare运算符，如果比较返回true，则返回所有
该列家族中的列。

#### QualifierFilter

This filter takes a compare operator and a comparator. It compares each qualifier name with the
comparator using the compare operator and if the comparison returns true, it returns all the keyvalues in that column.

该滤波器带有一个比较运算符和一个比较器。 它将每个限定符名称与
使用compare运算符的比较器，如果比较返回true，它将返回该列中的所有键值。

#### ValueFilter

This filter takes a compare operator and a comparator. It compares each value with the comparator
using the compare operator and if the comparison returns true, it returns that key-value.

该滤波器带有一个比较运算符和一个比较器。 它将每个值与比较器进行比较
使用比较运算符，如果比较返回true，则返回该键值。

#### DependentColumnFilter

This filter takes two arguments – a family and a qualifier. It tries to locate this column in each row
and returns all key-values in that row that have the same timestamp. If the row doesn’t contain the
specified column – none of the key-values in that row will be returned.

该过滤器接受两个参数-family 和限定词。 它尝试在每一行中定位此列
并返回该行中所有具有相同时间戳的键值。 如果该行不包含
指定的列-该行中的任何键值均不会返回。

#### SingleColumnValueFilter

This filter takes a column family, a qualifier, a compare operator and a comparator. If the specified
column is not found – all the columns of that row will be emitted. If the column is found and the
comparison with the comparator returns true, all the columns of the row will be emitted. If the
condition fails, the row will not be emitted.

该过滤器包含一个列族，一个限定符，一个比较运算符和一个比较器。 如果指定
找不到列-该行的所有列都将被发射。 如果找到该列并且
与比较器比较返回true时，将发出该行的所有列。 如果
条件失败，将不会发出该行。

#### SingleColumnValueExcludeFilter

This filter takes the same arguments and behaves same as SingleColumnValueFilter – however, if
the column is found and the condition passes, all the columns of the row will be emitted except for
the tested column value.

该过滤器采用相同的参数，其行为与SingleColumnValueFilter相同-但是，如果
找到该列并且条件通过后，将发射该行的所有列，除了
测试的列值。

#### ColumnRangeFilter

This filter is used for selecting only those keys with columns that are between minColumn and
maxColumn. It also takes two boolean variables to indicate whether to include the minColumn and
maxColumn or not.

该过滤器仅用于选择那些列在minColumn和之间的键
maxColumn。 它还需要两个布尔变量来指示是否包括minColumn和
是否使用maxColumn。