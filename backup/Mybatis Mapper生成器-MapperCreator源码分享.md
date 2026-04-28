来源：huzhiyong
```java
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

/**
 * Created by huzhiyong on 2017/2/7.
 * 用于生成基本的mapper文件
 * 输入：SQLyog 导出的建表语句
 * 输出：1，基本Dao和DO类，2，mapper文件，3,测试代码  全部输出到 output.txt 文件
 *
 * sql规范：
 * 1, 数据库字段名全小写，用下划线隔开
 * 2，默认每个表都有 id 字段（自增主键） modified_date
 * 3, 每列都要有注释，列注释当中不包括 英文逗号，数据库注释不包括 英文括号
 * 4，删除掉索引行，当有联合索引时，会报错
 */
public class MapperCreator {
    private static final HashMap<String,String> sqlJavaTypeMap = new HashMap<>();
    private static final HashMap<String,String> sqlJdbcTypeMap = new HashMap<>();
    static{
        sqlJavaTypeMap.put("bigint", "Long") ;
        sqlJavaTypeMap.put("int", "Integer") ;
        sqlJavaTypeMap.put("tinyint", "Integer") ;
        sqlJavaTypeMap.put("varchar", "String") ;
        sqlJavaTypeMap.put("text", "String") ;
        sqlJavaTypeMap.put("char", "String") ;
        sqlJavaTypeMap.put("datetime", "Date") ;
        sqlJavaTypeMap.put("timestamp", "Date") ;
        sqlJavaTypeMap.put("mediumtext", "String") ;

        sqlJdbcTypeMap.put("bigint", "BIGINT") ;
        sqlJdbcTypeMap.put("int", "INTEGER") ;
        sqlJdbcTypeMap.put("tinyint", "INTEGER") ;
        sqlJdbcTypeMap.put("varchar", "VARCHAR") ;
        sqlJdbcTypeMap.put("text", "VARCHAR") ;
        sqlJdbcTypeMap.put("char", "CHAR") ;
        sqlJdbcTypeMap.put("datetime", "TIMESTAMP") ;// 注意，这里使用这个类型，不能直接使用DATE,mybatis的Date只有年月日
        sqlJdbcTypeMap.put("timestamp", "TIMESTAMP") ;
        sqlJdbcTypeMap.put("mediumtext", "VARCHAR") ;
    }

    private static final String DAO_END = "Dao" ;
    private static final String DO_END = "DO" ;



    // 修改这两个参数
    private static final String OUTPUT_DIR = "C:\\Users\\jrliangbo\\Downloads" ;
    private static final String PACKAGE_NAME = "com.heytap.mall.comment.core.domain.dao." ;


    public static void main(String[] args) throws Exception{

        // 此处将sql建表语句粘出，注意sql规范
       String sql = "CREATE TABLE `t_comment_liking` (\n" +
               "  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id 自增主键',\n" +
               "  `comment_id` bigint(20) NOT NULL COMMENT '评论id 评论id',\n" +
               "  `user_id` varchar(64) DEFAULT NULL COMMENT '用户唯一标识 用户唯一标识',\n" +
               "  `device_id` varchar(64) DEFAULT NULL COMMENT '设备id',\n" +
               "  `liking` tinyint(1) DEFAULT NULL COMMENT '点赞 点赞与否，1点赞',\n" +
               "  `data_status` tinyint(1) DEFAULT NULL COMMENT '数据有效性 数据是否有效，1有效',\n" +
               "  `create_time` timestamp NOT NULL COMMENT '创建时间',\n" +
               "  `modify_time` timestamp NOT NULL COMMENT '修改时间',\n" +
               "  `created` varchar(32) NOT NULL DEFAULT '' COMMENT '创建人',\n" +
               "  `modified` varchar(32) NOT NULL DEFAULT '' COMMENT '更新人',\n" +
               "  PRIMARY KEY (`id`)\n" +
               ") ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='评论点赞表 '" ;


        sql = sql.replace("\n","") ;
        process(sql) ;
        System.out.println("end");

    }

    /**
     * 生成 java DO类
     * 生成 Dao类
     * 生成 mapper文件
     * 生成 测试类
     */
    private static void process(String allData){
        int begin = allData.indexOf("(");
        int end = allData.lastIndexOf(")");

        String firstLine = allData.substring(0, begin) ;
        String columnLine = allData.substring(begin+1,end);

        String tableName = getTableName(firstLine) ;

        List<Item> columnList = getColumnList(columnLine) ;

        String baseName = columnName2JavaName(tableName) ;
        String doVarName = baseName + DO_END ;
        String daoVarName = baseName + DAO_END ;
        String doClassName = doVarName.substring(0,1).toUpperCase() + doVarName.substring(1) ;
        String daoClassName = daoVarName.substring(0,1).toUpperCase() + daoVarName.substring(1) ;
        StringBuilder sb = new StringBuilder("") ;

        sb.append(doClassName);
        sb.append("-----------------------------------------------------------\n\n") ;
        sb.append(createDoFile(columnList)) ;
        sb.append("\n\n") ;


        sb.append(daoClassName);
        sb.append("-----------------------------------------------------------\n\n") ;
        sb.append(createDaoFile(doVarName, doClassName, daoClassName)) ;
        sb.append("\n\n") ;


        sb.append(baseName.substring(0,1).toUpperCase());
        sb.append(baseName.substring(1));
        sb.append("Mapper.xml-------------------------------------------------\n\n") ;
        sb.append(createMapperFile(columnList,daoClassName,doClassName,tableName));
        sb.append("\n\n");



        sb.append(daoClassName);
        sb.append("Test.java");
        sb.append("-----------------------------------------------------------\n\n") ;
        sb.append(createTestFile(columnList,doClassName,doVarName,daoClassName,daoVarName)) ;
        sb.append("\n\n") ;


        writeToFile(OUTPUT_DIR, sb.toString()) ;

        return ;
    }


    private static void writeToFile(String dir, String data){
        String filePath = dir + "\\\\" + "output.txt" ;
        FileWriter fos;
        try {
            fos = new FileWriter(filePath);
            fos.write(data);
            fos.flush();
            fos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }



    private static String createDoFile(List<Item> columnList){
        StringBuilder sb = new StringBuilder("") ;
        for(Item item : columnList){
            sb.append("/**  ");
            sb.append(item.getComment());
            sb.append("  */\n");

            sb.append("private ");
            sb.append(item.getJavaType());
            sb.append(" ");
            sb.append(item.getName()) ;
            sb.append(";\n\n") ;
        }

        sb.append("\n\n");
        return sb.toString() ;
    }

    private static String createDaoFile(String doVarName,String doClassName,String daoClassName){
        String base  = "@SuperDao\n" +
                "public interface TMPDAO {\n" +
                "    int insert(TMPDO tmpdo);\n" +
                "    int update(TMPDO tmpdo);\n" +
                "    int delete(Long id);\n" +
                "    TMPDO getById(Long id);\n" +
                "}";
        base = base.replace("TMPDAO",daoClassName);
        base = base.replace("TMPDO",doClassName);
        base = base.replace("tmpdo",doVarName);
        return base ;

    }

    private static String createMapperFile(List<Item> columnList,String daoClassName,String doClassName,String tableName){
        StringBuilder sb = new StringBuilder("<?xml version=\"1.0\" encoding=\"UTF-8\" ?>\n" +
                "<!DOCTYPE mapper PUBLIC \"-//mybatis.org//DTD Mapper 3.0//EN\" \"http://mybatis.org/dtd/mybatis-3-mapper.dtd\" >\n" +
                "<mapper namespace=\"");
        sb.append(PACKAGE_NAME);
        sb.append(daoClassName);
        sb.append("\">\n");

        sb.append("<resultMap id=\"BaseResultMap\" type=\"");
        sb.append(doClassName);
        sb.append("\">\n") ;

        sb.append("<id column=\"id\" property=\"id\" jdbcType=\"BIGINT\"/>\n") ;

        for(Item item : columnList){
            if(!item.getName().equals("id")){
                sb.append("<result column=\"");
                sb.append(item.getColumn());
                sb.append("\" property=\"");
                sb.append(item.getName());
                sb.append("\" jdbcType=\"");
                sb.append(item.getJdbcType());
                sb.append("\"/>\n");

            }
        }

        sb.append(" </resultMap>\n") ;

        sb.append("<sql id=\"Base_Column_List\">\n");
        sb.append(columnList.get(0).getColumn());
        for(int i=1; i<columnList.size(); i++){
            sb.append(",") ;
            sb.append(columnList.get(i).getColumn()) ;
        }
        sb.append("\n");
        sb.append("</sql>\n") ;

        // insert
        sb.append(createInsertSql(columnList, doClassName, tableName)) ;
        sb.append("\n\n");

        // update
        sb.append(createUpdateSql(columnList, doClassName, tableName)) ;
        sb.append("\n\n");

        // getById
        sb.append(createQuerySql(tableName)) ;
        sb.append("\n\n");

        // delete
        sb.append(createDeleteSql(tableName)) ;
        sb.append("\n\n");

        sb.append("</mapper>\n");
        return sb.toString() ;

    }


    private static String createInsertSql(List<Item> columnList,String doClassName,String tableName){
        StringBuilder sb = new StringBuilder("");
        sb.append("<insert id=\"insert\" useGeneratedKeys=\"true\" keyProperty=\"id\" parameterType=\"");
        sb.append(doClassName);
        sb.append("\">\n");
        sb.append("INSERT into ");
        sb.append(tableName) ;
        sb.append("\n");
        sb.append("<trim prefix=\"(\" suffix=\")\">\n");
        for(Item item : columnList){
            if(!item.getColumn().equals("id") && !item.getColumn().equals("modified_date")){
                if(item.getColumn().equals("created_date")){
                    sb.append("created_date,\n");
                }else if(item.isCanEmpty()){
                    sb.append("<if test=\"");
                    sb.append(item.getName());
                    sb.append("!= null\">\n");
                    sb.append(item.getColumn());
                    sb.append(",\n");
                    sb.append("</if>\n");
                }else{
                    sb.append(item.getColumn());
                    sb.append(",\n") ;
                }
            }
        }
        sb.append("modified_date\n");
        sb.append("</trim>\n") ;


        sb.append("<trim prefix=\"values (\" suffix=\")\">\n");
        for(Item item : columnList){
            if(!item.getColumn().equals("id") && !item.getColumn().equals("modified_date")){
                if(item.getColumn().equals("created_date")){
                    sb.append("current_timestamp,\n");
                }else if(item.isCanEmpty()){
                    sb.append("<if test=\"");
                    sb.append(item.getName());
                    sb.append(" != null\">\n");
                    sb.append("#{");
                    sb.append(item.getName());
                    sb.append(",jdbcType=");
                    sb.append(item.getJdbcType());
                    sb.append("},\n");
                    sb.append("</if>\n");
                }else{
                    sb.append("#{");
                    sb.append(item.getName());
                    sb.append(",jdbcType=");
                    sb.append(item.getJdbcType());
                    sb.append("},\n");
                }
            }
        }
        sb.append("current_timestamp\n");
        sb.append("</trim>\n") ;
        sb.append("</insert>\n") ;
        return sb.toString() ;
    }


    private static String createUpdateSql(List<Item> columnList,String doClassName,String tableName){
        StringBuilder sb = new StringBuilder("");
        sb.append("<update id=\"update\" parameterType=\"");
        sb.append(doClassName);
        sb.append("\">\n");
        sb.append("update ");
        sb.append(tableName);
        sb.append("\n");
        sb.append("<set>\n");


        for(Item item : columnList){
            if(!item.getColumn().equals("id")
                    && !item.getColumn().equals("modified_date")
                    && !item.getColumn().equals("created_date")){

                sb.append("<if test=\"");
                sb.append(item.getName());
                sb.append(" != null\">\n");
                sb.append(item.getColumn());
                sb.append(" = #{");
                sb.append(item.getName());
                sb.append(",jdbcType=");
                sb.append(item.getJdbcType());
                sb.append("},\n");
                sb.append("</if>\n");
            }
        }
        sb.append("modified_date = current_timestamp\n");
        sb.append("</set>\n");
        sb.append("where id = #{id,jdbcType=BIGINT}\n");
        sb.append("</update>\n");


        return sb.toString() ;
    }

    private static String createQuerySql(String tableName){
        StringBuilder sb = new StringBuilder("");
        sb.append("<select id=\"getById\" resultMap=\"BaseResultMap\" parameterType=\"java.lang.Long\">\n");
        sb.append("SELECT\n");
        sb.append("<include refid=\"Base_Column_List\" />\n");
        sb.append("from ");
        sb.append(tableName);
        sb.append("\n");
        sb.append("where id = #{id,jdbcType=BIGINT}\n");
        sb.append("</select>\n") ;

        return sb.toString() ;
    }

    private static String createDeleteSql(String tableName){
        StringBuilder sb = new StringBuilder("");
        sb.append("<delete id=\"delete\"  parameterType=\"java.lang.Long\">\n");
        sb.append("DELETE\n");
        sb.append("from ");
        sb.append(tableName);
        sb.append("\n");
        sb.append("where id = #{id,jdbcType=BIGINT}\n");
        sb.append("</delete>\n") ;

        return sb.toString() ;
    }

    private static String createTestFile(List<Item> columnList,String doClassName,String doVarName,String daoClassName, String daoVarName){

        String model = "public class TMPXXXDaoTest extends TestBase {\n" +
                "    private static final Logger logger = LoggerFactory.getLogger(TMPXXXDaoTest.class);\n" +
                "\n" +
                "    @Resource\n" +
                "    private TMPXXXDao tmpXXXDao;\n" +
                "\n" +
                "    private Long id ;\n" +
                "\n" +
                "    @Test\n" +
                "    public void testInsert(){\n" +
                "        TMPXXXDO tmpXXXDO = create();\n" +
                "        int result = tmpXXXDao.insert(tmpXXXDO);\n" +
                "        Assert.assertTrue(result > 0);\n" +
                "        id = tmpXXXDO.getId() ;\n" +
                "        Assert.assertNotNull(id);\n" +
                "        Assert.assertTrue(id > 0);\n" +
                "    }\n" +
                "\n" +
                "    @Test(dependsOnMethods = \"testInsert\")\n" +
                "    public void testGetById(){\n" +
                "        TMPXXXDO tmpXXXDO = tmpXXXDao.getById(id);\n" +
                "        Assert.assertNotNull(tmpXXXDO);\n" +
                "        logger.info(\"testGetById获得对象： {}\", tmpXXXDO);\n" +
                "    }\n" +
                "\n" +
                "    @Test(dependsOnMethods = \"testGetById\")\n" +
                "    public void testUpdate(){\n" +
                "        TMPXXXDO tmpXXXDO = create();\n" +
                "        tmpXXXDO.setId(id);\n" +
                "        SET_STRING_PLACEHOLDER\n" +
                "        int result = tmpXXXDao.update(tmpXXXDO);\n" +
                "        Assert.assertTrue(result > 0);\n" +
                "    }\n" +
                "\n" +
                "    @Test(dependsOnMethods = \"testUpdate\")\n" +
                "    public void testGetById2(){\n" +
                "        TMPXXXDO tmpXXXDO = tmpXXXDao.getById(id);\n" +
                "        Assert.assertNotNull(tmpXXXDO);\n" +
                "        logger.info(\"testGetById2获得对象： {}\", tmpXXXDO);\n" +
                "    }\n" +
                "\n" +
                "    @Test(dependsOnMethods = \"testGetById2\")\n" +
                "    public void testDelete(){\n" +
                "        int result = tmpXXXDao.delete(id);\n" +
                "        Assert.assertTrue(result > 0);\n" +
                "    }\n" +
                "\n" +
                "    @Test(dependsOnMethods = \"testDelete\")\n" +
                "    public void testGetById3(){\n" +
                "        TMPXXXDO tmpXXXDO = tmpXXXDao.getById(id);\n" +
                "        Assert.assertNull(tmpXXXDO);\n" +
                "    }\n" +
                "\n" +
                "\n" +
                "    private TMPXXXDO create(){\n" +
                "        TMPXXXDO tmpXXXDO = new TMPXXXDO();\n" +
                "        SET_STRING_PLACEHOLDER\n" +
                "        return tmpXXXDO ;\n" +
                "    }\n" +
                "}" ;

        model = model.replace("TMPXXXDao",daoClassName);
        model = model.replace("tmpXXXDao",daoVarName);
        model = model.replace("TMPXXXDO",doClassName);
        model = model.replace("tmpXXXDO",doVarName);
        String setString = getSetString(columnList,doVarName);
        model = model.replace("SET_STRING_PLACEHOLDER", setString);
        return model ;

    }

    private static String getSetString(List<Item> columnList,String doVarName){
        StringBuilder sb = new StringBuilder("");
        for(Item item:columnList){
            if(!item.getColumn().equals("id")){
                if(item.isCanEmpty()){
                    sb.append("//") ;
                }
                sb.append(doVarName);
                sb.append(".set");
                sb.append(item.getName().substring(0,1).toUpperCase());
                sb.append(item.getName().substring(1));
                sb.append("(");

                // 当有新的类型时，需要在此处添加
                if(item.getJavaType().equals("Integer")){
                    sb.append("0");
                }else if(item.getJavaType().equals("Long")){
                    sb.append("0L");
                }else if(item.getJavaType().equals("String")){
                    sb.append("\"hello world 测试\"");
                }else if(item.getJavaType().equals("Date")){
                    sb.append("new Date()");
                }else{
                    sb.append("");
                }

                sb.append(");\n");
            }

        }

        return sb.toString() ;

    }



    private static List<Item> getColumnList(String data){
        List<Item> result = new ArrayList<Item>();
        String[] array = data.split(",") ;
        for(String line : array){
            if(!isKeyLine(line)){
                Item item = new Item();
                item.setColumn(getColumn(line));
                item.setComment(getComment(line));
                item.setSqlType(getType(line));
                item.setCanEmpty(canEmpty(line));
                if(!sqlJavaTypeMap.containsKey(item.getSqlType())){
                    throw new RuntimeException("找不到对应类型") ;
                }
                item.setJavaType(sqlJavaTypeMap.get(item.getSqlType()));
                item.setJdbcType(sqlJdbcTypeMap.get(item.getSqlType()));
                item.setName(columnName2JavaName(item.getColumn()));
                result.add(item);
            }

        }
        return result ;
    }

    private static boolean isKeyLine(String line){
        line = line.trim();
        line = line.toUpperCase() ;
        if(line.startsWith("PRIMARY")
                || line.startsWith("KEY")
                || line.startsWith("INDEX")
                || line.startsWith("UNIQUE")
                ){
            return true ;
        }
        return false ;
    }



    private static String removeQuote(String data, int quote){
        data = data.trim() ;
        int begin = data.indexOf(quote);
        if(begin > -1){
            int end = data.indexOf(quote,begin+1);
            if(end > -1){
                return data.substring(begin+1,end) ;
            }
        }
        return data ;
    }

    private static String getTableName(String firstLine){
        String[] array = firstLine.split(" ");
        int length = array.length;
        return removeQuote(array[length-1],'`');
    }

    // 获取注释，COMMENT 之后第一个用 引号包起来的内容
    private static String getComment(String line){
        String[] array = line.split("COMMENT");
        return removeQuote(array[1],'\'') ;
    }

    private static String getColumn(String line){
        return removeQuote(line,'`');
    }

    private static boolean canEmpty(String line){
        // 有默认值的认为插入是可以为空
        if(line.contains("NOT NULL DEFAULT")){
            return true ;
        }
        return !(line.contains("NOT NULL")) ;
    }

    private static String getType(String line){
        line = line.trim() ;
        String[] array = line.split(" ");
        String name = array[1];
        int end = name.indexOf("(");
        if(end > 1){
            return name.substring(0,end) ;
        }
        return name ;
    }




    private static String columnName2JavaName(String column){
        column = column.toLowerCase() ;
        String[] array = column.split("_") ;
        StringBuilder sb = new StringBuilder("");
        sb.append(array[0]);
        for(int i=1; i<array.length; i++){
            sb.append(array[i].substring(0,1).toUpperCase()) ;
            sb.append(array[i].substring(1)) ;
        }

        return sb.toString() ;

    }


}

class Item{
    private String name; // java字段名
    private String column ; // 数据库列名
    private String sqlType ;
    private String javaType;
    private String jdbcType ;
    private boolean canEmpty ;// 是否可为Null
    private String comment ; //注释

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getColumn() {
        return column;
    }

    public void setColumn(String column) {
        this.column = column;
    }

    public String getSqlType() {
        return sqlType;
    }

    public void setSqlType(String sqlType) {
        this.sqlType = sqlType;
    }

    public String getJavaType() {
        return javaType;
    }

    public void setJavaType(String javaType) {
        this.javaType = javaType;
    }

    public String getJdbcType() {
        return jdbcType;
    }

    public void setJdbcType(String jdbcType) {
        this.jdbcType = jdbcType;
    }

    public boolean isCanEmpty() {
        return canEmpty;
    }

    public void setCanEmpty(boolean canEmpty) {
        this.canEmpty = canEmpty;
    }

    public String getComment() {
        return comment;
    }

    public void setComment(String comment) {
        this.comment = comment;
    }
}

```