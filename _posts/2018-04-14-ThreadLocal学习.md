---
title: ThreadLocal学习
categories:
 - Java
tags: 
 - Java
---

ThreadLocal简单的说就是线程本地变量，多线程环境下安全。
比如说多数据源，一个datasource的变量，应用根据datasource中配置的是user还是product来获取数据库连接。
那么就可以用ThreadLocal来存储。实现如下：
```
	private ThreadLocal<String> local = new ThreadLocal<String>();
	public void switchDataSource(String dataSourceName) {
		this.local.set(dataSourceName);
	}

	// 获取一个数据源配置
	public DataSourceConfig getDataSourceConfig() {
		String dataSourceName = this.local.get();
		DataSourceConfig dsc = new DataSourceConfig();
		DruidDataSource dataSource = null;
		if (dataSourceName == null || dataSourceName.trim().length() <= 0) {
			dataSource = ((DruidDataSource) defaultDataSource);
		} else {
			dataSource = (DruidDataSource) this.applicationContext.getBean(dataSourceName);
		}

		dsc.setUrl(dataSource.getUrl());
		dsc.setPassword(dataSource.getPassword());
		dsc.setUsername(dataSource.getUsername());
		dsc.setDriverClassName(dataSource.getDriverClassName());

		return dsc;
	}

	private DataSource getDataSource(){
		String dataSourceName = this.local.get();
		if (dataSourceName == null || dataSourceName.trim().length() <= 0) {
			return defaultDataSource;
		}
		try {
			return (DataSource) this.applicationContext.getBean(dataSourceName);
		} catch (NoSuchBeanDefinitionException ex) {
			logger.error(",getDataSource.error", ex);
			throw new DateSourceException(ResultCode.DATA_SOURCE_FAIL); 
		}
	}
```











