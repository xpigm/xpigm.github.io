---
layout: post
title:  Java异常：Not enough variable values available to expand 'var'
categories: exception
tags: [java, pit, experience]
no-post-nav: true
comments: true
---

# Java异常：Not enough variable values available to expand 'var'

当你使用 MockMvc 进行单元测试或者 使用 RestTemplate 访问 rest接口时

```java
@Test
public void testREST() throws Exception {
    mockMvc.perform(MockMvcRequestBuilders.get("/zj/getDatailOfTemplate?key={var}"))
        .andDo(MockMvcResultHandlers.print())
        .andExpect(MockMvcResultMatchers.status().isOk())
        .andDo(MockMvcResultHandlers.print())
        .andReturn();
}

```


很有可能会遇到类似这种错误，确切来说肯定会遇到这种错误
`Not enough variable values available to expand 'var'`

从错误信息来看，好像是var被当作一个变量了。
通过调试发现，会调用这么一段代码
```java
// Static expansion helpers

static String expandUriComponent(String source, UriTemplateVariables uriVariables) {
	if (source == null) {
		return null;
	}
	if (source.indexOf('{') == -1) {
		return source;
	}
	if (source.indexOf(':') != -1) {
		source = sanitizeSource(source);
	}
	Matcher matcher = NAMES_PATTERN.matcher(source);
	StringBuffer sb = new StringBuffer();
	while (matcher.find()) {
		String match = matcher.group(1);
		String variableName = getVariableName(match);
		Object variableValue = uriVariables.getValue(variableName);
		if (UriTemplateVariables.SKIP_VALUE.equals(variableValue)) {
			continue;
		}
		String variableValueString = getVariableValueAsString(variableValue);
		String replacement = Matcher.quoteReplacement(variableValueString);
		matcher.appendReplacement(sb, replacement);
	}
	matcher.appendTail(sb);
	return sb.toString();
}
```

已经很明显了，能用模式与 `\{([^/]+?)\}` 匹配的value都会被视为变量，因此 `{var}`被作为了变量（占位符），解析的时候发生了异常。



**解决方案**：
1. 使用 `URLEncoder` 对特殊字符进行编码

2. 定义一个包含特殊字符的`String`变量

   ```java
   String key = "{var}"
   ```

   用占位符放置在URL里

   ```java
    mockMvc.perform(MockMvcRequestBuilders.get("/zj/getDatailOfTemplate?key={key}"))
   ```

   ​
