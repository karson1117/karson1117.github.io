---
title: List集合按对象的某个字段去重
date: 2017-05-26 15:37:46
tags:
	- Java基础
categories:
	- 编程
	- Java
---
思路:
利用Set(集合)的不可重复性：List-->Set-->List
Set是最简单的一种集合。集合中的对象不按特定的方式排序，并且没有重复对象。 Set接口主要实现了两个实现类：
●HashSet：HashSet类按照哈希算法来存取集合中的对象，存取速度比较快。 
●TreeSet：TreeSet类实现了SortedSet接口，能够对集合中的对象进行排序。
<!--more-->
### 示例代码
set集合的构造方式列出来了三种:new Comparator; lambda表达式; Comparator.comparing(); 
{% codeblock %}
public class CompareTest {

		public static void main(String[] args) {
	
	    	List<CompanyMsgParam> companyList = new ArrayList<>();
	    	CompanyMsgParam c1 = new CompanyMsgParam("2", "001", "A公司", "0");
	    	CompanyMsgParam c2 = new CompanyMsgParam("1", "001", "B公司", "1");
	    	CompanyMsgParam c3 = new CompanyMsgParam("2", "002", "C公司", "2");
	    	CompanyMsgParam c4 = new CompanyMsgParam("1", "003", "D公司", "3");
	    	companyList.add(c1);
	    	companyList.add(c2);
	    	companyList.add(c3);
	    	companyList.add(c4);
	//第一种
	    	Set<CompanyMsgParam> set = new TreeSet<CompanyMsgParam>(new Comparator<CompanyMsgParam>() {
	        @Override
	        public int compare(CompanyMsgParam com1, CompanyMsgParam com2) {
	            return com1.getCompanyCode().compareTo(com2.getCompanyCode());
	       		}
	    	});
	//第二种
	    	Set<CompanyMsgParam> set = new TreeSet<CompanyMsgParam>((com1, com2) -> com1.getCompanyCode().compareTo(com2.getCompanyCode()));
	//第三种
	    	Set<CompanyMsgParam> set = new TreeSet<CompanyMsgParam>(Comparator.comparing(CompanyMsgParam::getCompanyCode));
	    	set.addAll(companyList);
	
	    	companyList = new ArrayList<CompanyMsgParam>(set);
	
	    	for (CompanyMsgParam param : companyList) {
	        	System.out.println(param.getCompanyCode());
	    		}
		}
	}

{% endcodeblock %}
### IDEA配置LanguageLevel和JavaCompiler版本的问题
按个人需要修改下面的配置即可：
{% codeblock %}
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.3.2</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
{% endcodeblock %}