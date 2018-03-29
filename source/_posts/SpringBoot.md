---
title: SpringBoot
author: mustard
date: 2018-03-26 12:16:01
tags:
  - IDEA
  - SpringBoot
categories:
  - 技术
---

## Springboot的使用方法

### 流程

自动建数据库实体表

使用控制器来判断进入那个页面,并且返回数据,使用DAO层对数据库进行自定义设计,更好的服务控制器

![](https://vgy.me/aMdInN.png)

### 自动创建数据库表

- 在`class`中添加注解

  ```java
  import javax.persistence.Entity;
  import javax.persistence.GeneratedValue;
  import javax.persistence.GenerationType;
  import javax.persistence.Id;
  import javax.persistence.Table;
  @Entity
  //声明这是一个实体类,要和数据库中的表对应
  @Table(name = "StuMess")
  //数据库中表的名称为StuMess
  public class StuMess {
      @Id
      @GeneratedValue(strategy= GenerationType.AUTO)
      //创建一个自增ID作为主键
      private Long id;
      private String name;
        public StuMess() {
      }

      public String getName() {
          return name;
      }

      public StuMess setName(String name) {
          this.name = name;
          return this;
      }
  }
  ```

- 刚刚只是声明了一个实体,接下来还需要对数据库进行设置,在`application.yml`中进行设置

  ```properties
  spring:
    thymeleaf:
      prefix: classpath:/templates/
      suffix: .html
      check-template-location: true
      mode: HTML5
    datasource:
      url: jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8&useSSL=false
      username: root
      password: root
      sql-script-encoding: utf-8
      #设置编码,一定要保证数据库的编码是utf8,不然可能会出现中文乱码问题
    jpa:
      database: mysql
      show-sql: true
      hibernate:
        ddl-auto: update
        #数据库自动更新
  ```


- 在DAO层实现自定义的`sql`语句

  ```java
    @Repository
    @Table(name="StuMess")
    //@Qualifier("StuMessRep")
    public interface StuMessDAO extends CrudRepository<StuMess, Long >{

        @Query("select stu from StuMess stu where stu.name =:name")
        public StuMess findStuName(@Param("name") String name);

        @Query("select stu from StuMess stu where stu.num =:num")
        public StuMess findStuNum(@Param("num") String num);


        @Query(value="select class_name,AVG(chinese),AVG(english),AVG(math),AVG(science),AVG(society) from stu_mess where class_name like :class_name group by class_name",nativeQuery=true)
        public List<Object> getAverage(@Param("class_name") String className);

        @Query(value="select class_name,MAX(chinese),MAX(english),MAX(math),MAX(science),MAX(society) from stu_mess where class_name like :class_name group by class_name",nativeQuery=true)
        public List<Object> getMax(@Param("class_name") String className);

        @Query(value = "SELECT class_name,chinese,english,math,science,society,name,num FROM test.stu_mess where name like :name",nativeQuery = true)
        public List<Object> getMesByname(@Param("name") String name);
    //简单的操作语句Springboot已经实现了,自己写的语句后要添加字段nativeQuery = true
    }

  ```

  ​

### 控制器与html

- 创建一个`package`为`controller`

  ```java
  @Controller
  public class IndexController {
  	String name = "";
  	@Autowired
      private StuMessDAO stuDao;
  	@RequestMapping("/index")
  	public String page3(Model model, String name) {
  		model.addAttribute("name", name);
  		return "index";
  	}
      //实现异步操作
      @RequestMapping("/echarts")
  	public String page1(Model model,String key){
  		return "echarts";
          //这里有一个问题,在下面的函数中无法得到key的值,所以考虑设置一个变量
  	}
  	@RequestMapping("/getEchartsData")
  	public @ResponseBody NetResult page1(String key){
  		NetResult r = new NetResult();
  		if(key == null){
  			key = "%%";
  		}else{
  			key = "%"+key+"%";
  		}
  		Object[] objects =(Object[])stuDao.getAverage(key).get(0);
  		System.out.println(objects[1]);
  		r.result = objects;
  //		model.addAttribute("host",objects[1]);
  		return r;
  	}
  }
  ```

- 异步操作的html,页面是放在`resources/templates`中的,`Springboot`会去自动找页面的

  ```html
  <!DOCTYPE html>
  <html lang="en" xmlns:th="http://www.thymeleaf.org">
      <meta charset="utf-8">
      <title>ECharts</title>
  </head>
  <body>
  <div id="main" style="height:400px"></div>
  <!-- ECharts单文件引入 -->
  <script src="/echarts.min.js"></script>
  <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>

  <script type="text/javascript" >

      $(document).ready(function(){
          var myChart = echarts.init(document.getElementById('main'));
          jQuery.ajax({
              type: 'POST',
              url: "/getEchartsData",
              dataType: 'json',
              success: function(json) {
                  if(json.status==0){
                      console.log(json);
                      var y_data=[];
                      for(var i = 1;i<json.result.length;i++){
                          y_data[i-1]=json.result[i];
                      }
                      var option = {
                          title: {
                              text: '班级均分'
                          },
                          tooltip: {},
                          legend: {
                              data:['得分']
                          },
                          xAxis: {
                              data: ["语文","英语","数学","科学","社会"]
                          },
                          yAxis: {},
                          series: [{
                              name: '均分',
                              type: 'bar',
                              data: y_data
                          }]
                      };
                      myChart.setOption(option);
                  }
              }
          });

      });

  </script>
  </body>
  </html>
  ```

  ​

## Q&A

### 从控制器中传数据至页面

- 在控制器中使用`model`

  ```java
  	@RequestMapping("/table")
  	public String page4(Model model, String name) {
  		List<StuMess> list_stu = stuDao.getStuMess();
          // 将数据放入model中，在页面可以使用th：each进行循环输出
         	model.addAttribute("list_stu", list_stu);
  		return "table_advanced";
  	}
  ```

- 前台页面

  ```html
  <!DOCTYPE html>
  <!-- 引入th标签 -->
  <html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <meta charset="UTF-8">
      <title>table</title>
  </head>
  <body>
  <table>
      <!-- 使用${}获得放入model中的值，list为临时变量，用于存储model中的数据 -->
      	<tr th:each="list : ${list_stu}">
              <td th:text="${list.name}"/>
              <td th:text="${list.num}"/>
              <td><a th:text="${list.className}" th:href="@{/chart}"></a></td>
              <td th:text="${list.chinese}"/>
              <td th:text="${list.math}"/>
              <td th:text="${list.english}"/>
              <td th:text="${list.science}"/>
              <td th:text="${list.society}"/>
              <!-- 执行超链接，进行新的请求并携带参数 -->
              <td><a th:href="@{/chartbyid?(key=${list.num})}">查看个人详细评价</a>			    </tr>
  </table>
  </body>
  </html>
  ```

