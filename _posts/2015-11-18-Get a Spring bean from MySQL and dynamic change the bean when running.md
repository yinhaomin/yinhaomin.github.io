---
layout: post
title: Get a Spring bean from MySQL and dynamic change the bean when running
author: "Yin Haomin"
date: 2015-11-18 12:00:00
tags:
    - Get bean from Spring
    - Spring
    - Dynamic change bean
    - Bean
---

如何实现从MuSQL中获取配置生成bean并动态的更改Bean?大体的步骤如下:

#### 1. 从DB中生成相应的Bean

将初始化propertyPlaceholder

```
	<bean id="propertyConfigurer23"
		class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="order" value="23" /> 
		<property name="properties" ref="dataBaseProperties" />
	</bean>

	<bean id="dataBaseProperties"
		class="com.baidu.hui.biz.database.properties.DatabaseProperties">
		<constructor-arg type="com.baidu.hui.dao.dao.CrawlerTemplateDAO"
			ref="crawlerTemplateDAO" />
		<constructor-arg type="com.baidu.hui.biz.utils.SpringContextUtil" ref="SpringContextUtil" />
	</bean>
```

使用初始化后的propertyPlaceholder设置一个bean

```
	<bean id="yihaodianMobileTemplate" class="com.baidu.hui.biz.vo.UrlTemplate"
		lazy-init="true">
		<property name="id">
			<value type="java.lang.Long">${yihaodianMobileTemplate_template_id}</value>
		</property>
		<property name="ucId">
			<value type="java.lang.Long">${yihaodianMobileTemplate_uc_id}</value>
		</property>
		<property name="domain">
			<value>${yihaodianMobileTemplate_domain}</value>
		</property>
		<property name="merchant">
			<value>${yihaodianMobileTemplate_merchant}</value>
		</property>
		<property name="secendLevelDomain">
			<value>${yihaodianMobileTemplate_second_level_domain}</value>
		</property>
		<property name="type">
			<value type="java.lang.Integer">${yihaodianMobileTemplate_crawl_type}</value>
		</property>
		<property name="outIdPatten">
			<value type="java.lang.String">${yihaodianMobileTemplate_out_id_patten}</value>
		</property>
		<property name="pattenIndex">
			<value type="java.lang.Integer">${yihaodianMobileTemplate_pattern_index}</value>
		</property>
		<property name="htmlCharset">
			<value>${yihaodianMobileTemplate_html_charset}</value>
		</property>
		<property name="updateTime">
			<value type="java.lang.Long">${yihaodianMobileTemplate_updatetime}</value>
		</property>
		<property name="xpaths">
			<map key-type="com.baidu.hui.biz.constance.HtmlAttribute">
				<entry key="TITLE">
					<value>${yihaodianMobileTemplate_title}</value>
				</entry>
				<entry key="PRICE">
					<value>${yihaodianMobileTemplate_price}</value>
				</entry>
				<entry key="IMAGES">
					<value>${yihaodianMobileTemplate_images}</value>
				</entry>
				<entry key="VALUE">
					<value>${yihaodianMobileTemplate_value}</value>
				</entry>
				<entry key="BRAND">
					<value>${yihaodianMobileTemplate_bean_name}</value>
				</entry>
				<entry key="STOCK">
					<value>${yihaodianMobileTemplate_bean_name}</value>
				</entry>
				<entry key="CATEGORY">
					<value>${yihaodianMobileTemplate_bean_name}</value>
				</entry>
				<entry key="SUBCATEGORY">
					<value>${yihaodianMobileTemplate_bean_name}</value>
				</entry>
				<entry key="THIRDCATEGORY">
					<value>${yihaodianMobileTemplate_bean_name}</value>
				</entry>
				<entry key="FOURTHCATEGORY">
					<value>${yihaodianMobileTemplate_bean_name}</value>
				</entry>
				<entry key="DESCRIPTION">
					<value>${yihaodianMobileTemplate_bean_name}</value>
				</entry>
			</map>
		</property>
	</bean>
```

#### 2. 更新容器内的template bean的属性
```
@Log4j
@Component
public class ReloadTemplateBeanManager implements ApplicationContextAware {

    @Autowired
    private ApplicationContext applicationContext;

    @Autowired
    private DatabaseProperties databaseProperties;

    @Autowired
    DefaultListableBeanFactory beanFactory;

    @Autowired
    private RedisFactory redisFactory;

    @Autowired
    @Qualifier(value = "jsonMessageSerializer")
    private MessageSerializer messageSerializer;

    @Autowired
    private CrawlerTemplateDAO crawlerTemplateDAO;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    /**
     * 更新容器内的template bean的属性
     */
    public void refreshContext() {
        try {
            databaseProperties.afterPropertiesSet();
        } catch (Exception e) {
            log.error("Error in databaseProperties.afterPropertiesSet(): ", e);
        }

        Properties properties = databaseProperties.getProperties();

        for (String beanName : CrawlTemplateConstant.CRAWL_TEMPLATE_NAMES) {
            UrlTemplate urlTemplate = null;
            try {
                urlTemplate = (UrlTemplate) ((ConfigurableApplicationContext) applicationContext).getBean(beanName);
                String keyPrefix = CrawlTemplateConstant.DEFAULT_TEMPLATE_NAME;
                if (beanName.equals(CrawlTemplateConstant.JD_PC_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.JD_PC_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.AMAZON_DP_PC_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.AMAZON_DP_PC_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.DD_PC_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.DD_PC_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.YHD_PC_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.YHD_PC_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.GM_PC_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.GM_PC_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.SN_PC_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.SN_PC_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.WPH_PC_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.WPH_PC_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.DEFAULT_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.DEFAULT_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.YHD_TUAN_PC_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.YHD_TUAN_PC_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.GM_TUAN_PC_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.GM_TUAN_PC_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.GM_Q_PC_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.GM_Q_PC_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.YX_PC_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.YX_PC_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.AMZON_GP_PC_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.AMZON_GP_PC_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.GM_ITEM_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.GM_ITEM_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.JD_MOBILE_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.JD_MOBILE_TEMPLATE_NAME;
                } else if (beanName.equals(CrawlTemplateConstant.YHD_MOBILE_TEMPLATE_NAME)) {
                    keyPrefix = CrawlTemplateConstant.YHD_MOBILE_TEMPLATE_NAME;
                }
                try {
                    urlTemplate.setId(Long.parseLong(properties.get(keyPrefix + CrawlTemplateConstant.TEMPLATE_ID)
                            .toString()));
                    urlTemplate.setDomain(properties.get(keyPrefix + CrawlTemplateConstant.DOMAIN).toString());
                    urlTemplate.setMerchant(properties.get(keyPrefix + CrawlTemplateConstant.MERCHANT).toString());
                    urlTemplate.setSecendLevelDomain(properties.get(
                            keyPrefix + CrawlTemplateConstant.SECOND_LEVEL_DOMAIN).toString());
                    Map<HtmlAttribute, String> xpaths = generateXpath(keyPrefix, properties);
                    urlTemplate.setXpaths(xpaths);
                    urlTemplate.setOutIdPatten(properties.get(keyPrefix + CrawlTemplateConstant.OUT_ID_PATTERN)
                            .toString());
                    urlTemplate.setPattenIndex(Integer.parseInt(properties.get(
                            keyPrefix + CrawlTemplateConstant.PATTERN_INDEX).toString()));
                    urlTemplate.setHtmlCharset(properties.get(keyPrefix + CrawlTemplateConstant.HTML_CHARSET)
                            .toString());
                    urlTemplate.setUpdateTime(System.currentTimeMillis());
                } catch (Exception e) {
                    log.error("Failed to set properties:", e);
                }
            } catch (Exception e) {
                log.error("Error in reolad:", e);
            }
        }
        // ((ConfigurableApplicationContext) applicationContext).refresh();
    }

    /**
     * 为template生成xpath
     * 
     * @param keyPrefix
     * @param properties
     * @return
     */
    private Map<HtmlAttribute, String> generateXpath(String keyPrefix, Properties properties) {
        Map<HtmlAttribute, String> map = new HashMap<HtmlAttribute, String>();
        try {
            map.put(HtmlAttribute.TITLE, properties.getProperty(keyPrefix + CrawlTemplateConstant.TITLE));
        } catch (Exception e) {
            log.error("Failed to get the property:", e);
        }
        try {
            map.put(HtmlAttribute.PRICE, properties.getProperty(keyPrefix + CrawlTemplateConstant.PRICE));
        } catch (Exception e) {
            log.error("Failed to get the property:", e);
        }
        try {
            map.put(HtmlAttribute.IMAGES, properties.getProperty(keyPrefix + CrawlTemplateConstant.IMAGES));
        } catch (Exception e) {
            log.error("Failed to get the property:", e);
        }
        try {
            map.put(HtmlAttribute.VALUE, properties.getProperty(keyPrefix + CrawlTemplateConstant.VALUE));
        } catch (Exception e) {
            log.error("Failed to get the property:", e);
        }
        try {
            map.put(HtmlAttribute.BRAND, properties.getProperty(keyPrefix + CrawlTemplateConstant.BRAND));
        } catch (Exception e) {
            log.error("Failed to get the property:", e);
        }
        try {
            map.put(HtmlAttribute.STOCK, properties.getProperty(keyPrefix + CrawlTemplateConstant.STOCK));
        } catch (Exception e) {
            log.error("Failed to get the property:", e);
        }
        try {
            map.put(HtmlAttribute.CATEGORY, properties.getProperty(keyPrefix + CrawlTemplateConstant.CATEGORY));
        } catch (Exception e) {
            log.error("Failed to get the property:", e);
        }
        try {
            map.put(HtmlAttribute.SUBCATEGORY, properties.getProperty(keyPrefix + CrawlTemplateConstant.SUB_CATEGORY));
        } catch (Exception e) {
            log.error("Failed to get the property:", e);
        }
        try {
            map.put(HtmlAttribute.THIRDCATEGORY, properties.getProperty(keyPrefix
                    + CrawlTemplateConstant.THIRD_CATEGORY));
        } catch (Exception e) {
            log.error("Failed to get the property:", e);
        }
        try {
            map.put(HtmlAttribute.FOURTHCATEGORY, properties.getProperty(keyPrefix
                    + CrawlTemplateConstant.FORTH_CATEGORY));
        } catch (Exception e) {
            log.error("Failed to get the property:", e);
        }
        try {
            map.put(HtmlAttribute.DESCRIPTION, properties.getProperty(keyPrefix + CrawlTemplateConstant.DESCRIPTION));
        } catch (Exception e) {
            log.error("Failed to get the property:", e);
        }

        return map;
    }
}

```

```
@Log4j
@Component
public class RefreshTemplateTimeManager {

    @Autowired
    private RedisFactory redisFactory;

    @Autowired
    @Qualifier(value = "jsonMessageSerializer")
    private MessageSerializer messageSerializer;

    @Autowired
    private CrawlerTemplateDAO crawlerTemplateDAO;
    
    @Autowired
    private SpringContextUtil springContextUtil;

    public void setRefreshTimeStamp() {
        JdbcTemplate template = (JdbcTemplate) springContextUtil.getBean("icHuiJdbcTemplate");
        long keyTimeStamp = crawlerTemplateDAO.getLatestTimestamp(template);
        Jedis jedis = null;
        try {
            jedis = redisFactory.getClient();
            jedis.set(CrawlTemplateConstant.HUI_CRAWL_TEMPLATE_UPDATETIME, keyTimeStamp + "");
        } catch (Exception e) {
            log.error(e);
        }
        try {
            redisFactory.returnResource(jedis);
        } catch (Exception e) {
            log.error(e);
        }
    }

}
```

