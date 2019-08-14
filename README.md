<h1 align="center">
  sql-page-helper
</h1>

## 简介

在分页机制中最重要的部分就是针对不同的数据库如何获取记录总数和分页 sql 的拼接问题，该工具就是为解决以上两个问题编写的，该项目已经集成在 _sunrise-spring-boot_ [[Gitee]](https://gitee.com/sunrise-chang/sunrise-spring-boot) [[GitHub]](https://github.com/SunRisingChang/sunrise-spring-boot) 项目中。

## 活动圈

- QQ 技术交流群 [678251003]
- 邮箱[Sun_Rising_Chang@hotmail.com]

## 前序准备

因为该项目并没有发布到 maven 仓库所以需手动 clone 到本地并引用它，本项目是由 [Maven](http://maven.apache.org/)构建。

## 功能

```bash
- 获取记录总数SQL语句

- 获取分页SQL语句

```

## 使用

这里只给出 spring 的使用方法

1、配置文件

```java
/**
 * 分页工具初始化类
 *
 * @author Sun Rising
 * @date 2019.05.29 02:04:04
 *
 */
@Configuration
public class DataPageConfig {

	private Logger logger = LoggerFactory.getLogger(DataPageConfig.class);

	@Autowired
	public JdbcTemplate jdbcTemplate;

	/**
	 * 初始化全局数据库分页处理器
	 *
	 * @author Sun Rising
	 * @date 2019.05.29 10:15:01
	 * @return
	 *
	 */
	@Bean
	public DataBaseHandle DataBaseFactory() {
		try {
			// 获取当前数据库类型
			String dbType = jdbcTemplate.getDataSource().getConnection().getMetaData().getDatabaseProductName();
			DataBaseType dbTypeString = DataBaseType.findTypeEnumByKey(dbType);
			DataBaseHandle dataBaseHandle = DataPageUtils.getDataPageUtils(dbTypeString);
			return dataBaseHandle;
		} catch (SQLException e) {
			logger.error("[DataPageConfig]未获取到数据源类型. -[" + this.getClass());
			throw new RuntimeException(e.getMessage());
		} catch (Exception e) {
			logger.error("[DataPageConfig]未找到支持的数据库分页处理器. -[" + this.getClass());
			throw new RuntimeException(e.getMessage());
		}
	}
}

```

2、查询方法定义

```java

	// 数据库SQL分页工具
	@Autowired
	private DataBaseHandle pageUtils;

	/**
	 * 分页方法 返回数据库原生字段集合
	 *
	 * @author Sun Rising
	 * @date 2019.05.29 02:26:53
	 * @param sql
	 * @return
	 *
	 */
	public PageInfo sqlPage(final String sql) {
		return sqlPage(sql, new ColumnMapRowMapper());
	}

	/**
	 * 分页方法 返回指定实体集合
	 *
	 * @author Sun Rising
	 * @date 2019.06.26 04:53:18
	 * @param sql
	 * @param rowMapper
	 * @return
	 *
	 */
	public <T> PageInfo sqlPage(final String sql, RowMapper<T> rowMapper) {
		// 数据页封装类
		PageInfo pageInfo = new PageInfo();
		// 先装配数据页封装类，pageUtils.getPageSql会读取
		try {
			BeanUtils.copyProperties(pageInfo, SpringWebUtils.getInterActionRequestsMap());
		} catch (Exception e) {
			logger.error("[DataPage]分页数据 copyProperties 失败. -[" + this.getClass());
		}
		// 获取总页数SQL
		String totalRecordsSql = pageUtils.getTotalRecords(sql);
		// 查询总页数
		int totalRecords = jdbcTemplate.queryForObject(totalRecordsSql, Integer.class);
		// 设置总页数
		pageInfo.setTotalRecords(totalRecords);
		// 获取分页SQL
		String pageSql = pageUtils.getPageSql(sql, pageInfo);
		// 执行查询
		List<T> provSends = jdbcTemplate.query(pageSql, rowMapper);
		// 设置返回结果集
		pageInfo.setDataBody(provSends);
		return pageInfo;
	}


```

3、查询示例

```java

	/**
	 * 查询Quartz日志
	 *
	 * @author Sun Rising
	 * @date 2019.07.03 10:05:58
	 * @return
	 *
	 */
	public PageInfo queryLogQuartz() {
		final String sqlStr = getFreemarkerUtils().getContextTemplate(this.getClass().getResourceAsStream("LogQuartzQuery.sql"));
		return sqlPage(sqlStr, new BeanPropertyRowMapper<LogQuartz>(LogQuartz.class));
	}

```

## License

[MIT](LICENSE)

Copyright (c) 2019-present SunRise
