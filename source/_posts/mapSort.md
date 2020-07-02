---
title: Map按值排序
date: 2017-03-15 13:37:37
tags:
	- java
	- note
categories:
	- Java基础
---
### Map按值排序
问题描述如下：
>现有Map<String,String>结构的数据，需要对其中包含的date值按降序排列输出
>直观了解：
>{
>"zhanshan" :"{"date":"2010-03-09 17:52:49:074","age":"21"},
>"lisi" :"{"date":"2015-01-09 10:52:49:088","age":"19"},
>"zhaowu" :"{"date":"2016-06-01 17:52:49:574","age":"30"}
>}
>注：此Map的值也为String类型

程序代码如下：
<!--more-->
{% codeblock %}
public Object MapSortTest(Map<String, String> map) {

	List<Map<String,String>> resList=new ArrayList<>();
	  if(!MapUtils.isEmpty(map)){
	
	//取Map映射集，遍历时可以getKey()，getValue()
	    List<Map.Entry<String,String>> entryList=new ArrayList<Map.Entry<String,String>>(map.entrySet());
	//自定义比较器
	    Collections.sort(entryList, new Comparator<Map.Entry<String, String>>(){
	        @Override
	        public int compare(Map.Entry<String, String> entry1, Map.Entry<String, String> entry2) {
	
	             String str1=entry1.getValue();
	             String str2=entry2.getValue();
		//切割截取date的值，subString前闭后开[)
	             String date1=str1.substring(str1.indexOf(":")+2,str1.indexOf(",")-1);
	             String date2=str2.substring(str2.indexOf(":")+2,str2.indexOf(",")-1);
	               try {
	                  Date d1=DateUtils.parseDate(date1,"yyyy-MM-dd HH:mm:ss:SSS");
	                  Date d2=DateUtils.parseDate(date2,"yyyy-MM-dd HH:mm:ss:SSS");
			//日期降序
	                  return d2.compareTo(d1);
	                } catch (ParseException e) {
	                    e.printStackTrace();
	                    return 0;
	                }
	            }
	        });
	     //遍历排序后的映射集，封装返回
	        Iterator<Map.Entry<String,String>> iter=entryList.iterator();
	        Map.Entry<String,String> temEntry =null;
	        while (iter.hasNext()){
	            Map<String,String> sortedMap = new LinkedHashMap<String,String>();
	            temEntry=iter.next();
	            sortedMap.put("type",temEntry.getKey());
	            sortedMap.put("message",temEntry.getValue());
	            resList.add(sortedMap);
	        }
	    }
	    return resList;
	}

{% endcodeblock %}
说明:上述程序不直接返回sortedMap，而是将其放入List中，是为了便于解析及防止架构上可能的重新排序（若返回map，可能会按map的key排序）。