---
title: "调整Spring HandlerMapping的顺序"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"Spring"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> "# 调整Spring HandlerMapping的顺序"

当请求形如：`/opendoc/jquery-1.10.2.min.js` 的静态资源时，如果恰好存在匹配这个请求的`Controller`时，默认情况下，这个静态资源请求会被 `RequestMappingHandlerMapping` 分配给这个`Controller`处理，从而可能找不到静态资源，例如存在下面这样的`Controller`：

```java
    @RequestMapping(value = "/{name}/{version}", method = {RequestMethod.POST, RequestMethod.GET})
    public void rest3(@PathVariable("name") String name, @PathVariable("version") String version,
                      HttpServletRequest request, HttpServletResponse response) {
        this.doRest(name, version, request, response);
    }
```

为了能正确匹配的静态资源，我们可以调整Spring `HandlerMapping`的顺序，让`SimpleUrlHandlerMapping`先于`RequestMappingHandlerMapping`去匹配请求，`SimpleUrlHandlerMapping`会返回一个用于加载静态资源的`ResourceHttpRequestHandler`

默认情况下`RequestMappingHandlerMapping` 先于`SimpleUrlHandlerMapping`匹配请求，`DispatcherServlet`中初始化`HandlerMapping`的顺序源码如下所示：

```java

	/**
	 * Initialize the HandlerMappings used by this class.
	 * <p>If no HandlerMapping beans are defined in the BeanFactory for this namespace,
	 * we default to BeanNameUrlHandlerMapping.
	 */
	private void initHandlerMappings(ApplicationContext context) {
		this.handlerMappings = null;

		if (this.detectAllHandlerMappings) {
			// Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
			Map<String, HandlerMapping> matchingBeans =
					BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
			if (!matchingBeans.isEmpty()) {
				this.handlerMappings = new ArrayList<>(matchingBeans.values());
				// We keep HandlerMappings in sorted order.
				AnnotationAwareOrderComparator.sort(this.handlerMappings);
			}
		}
		else {
			try {
				HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
				this.handlerMappings = Collections.singletonList(hm);
			}
			catch (NoSuchBeanDefinitionException ex) {
				// Ignore, we'll add a default HandlerMapping later.
			}
		}

		// Ensure we have at least one HandlerMapping, by registering
		// a default HandlerMapping if no other mappings are found.
		if (this.handlerMappings == null) {
			this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
			if (logger.isTraceEnabled()) {
				logger.trace("No HandlerMappings declared for servlet '" + getServletName() +
						"': using default strategies from DispatcherServlet.properties");
			}
		}
	}
```

默认情况下`RequestMappingHandlerMapping` 先于`SimpleUrlHandlerMapping`匹配请求：

![image-20210620131115664](https://picgo.6and.ltd/img/image-20210620131115664.png)



调整`HandlerMapping`顺序：

```java
 @Configuration
 public class WebConfiguration extends WebMvcConfigurationSupport {

     @Override
     public void addResourceHandlers(ResourceHandlerRegistry registry){
         registry.addResourceHandler("/opendoc/**").addResourceLocations("/opendoc/");
         registry.setOrder(0);
     }

     @Override
     @Bean
     public RequestMappingHandlerMapping requestMappingHandlerMapping(ContentNegotiationManager contentNegotiationManager, FormattingConversionService conversionService, ResourceUrlProvider resourceUrlProvider) {
         RequestMappingHandlerMapping handler =  super.requestMappingHandlerMapping(contentNegotiationManager, conversionService, resourceUrlProvider);
         handler.setOrder(1);
         return handler;
     }
 }
```

order值越小，优先级越高。

调整之后的HandlerMapping顺序：

![image-20210620130706589](https://picgo.6and.ltd/img/image-20210620130706589.png)

此时，`SimpleUrlHandlerMapping`会返回一个用于加载静态资源的`ResourceHttpRequestHandler`



#### 参考链接🔗：

[Spring - Set HandlerMapping Priority](https://stackoverflow.com/questions/17374549/spring-set-handlermapping-priority)

