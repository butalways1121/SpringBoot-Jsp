# SpringBoot-Jsp
SpringBoot整合Jsp
---
本文内容为SpringBoot与Jsp的整合,最终的成果是在[DeptManage项目](https://github.com/butalways1121/Spring-Boot-SpringMVC-MyBaits)的基础上，结合简单的Jsp页面整合成[DeptManageJsp项目](https://github.com/butalways1121/SpringBoot-Jsp),在Jsp页面实现对数据库的增删改查操作。主要内容如下：
<!-- more -->

1.在pom.xml文件中添加Jsp及Tomcat的相关依赖，如下：
```bash
<!--JSP 依赖  -->
  		<!-- servlet依赖. -->
		<dependency>
            <groupId>javax.servlet.jsp.jstl</groupId>
            <artifactId>jstl-api</artifactId>
            <version>1.2</version>
            <exclusions>
                <exclusion>
                    <groupId>javax.servlet</groupId>
                    <artifactId>servlet-api</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>javax.servlet.jsp</groupId>
                    <artifactId>jsp-api</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
 
        <dependency>
            <groupId>org.glassfish.web</groupId>
            <artifactId>jstl-impl</artifactId>
            <version>1.2</version>
            <exclusions>
                <exclusion>
                    <groupId>javax.servlet</groupId>
                    <artifactId>servlet-api</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>javax.servlet.jsp</groupId>
                    <artifactId>jsp-api</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>javax.servlet.jsp.jstl</groupId>
                    <artifactId>jstl-api</artifactId>
                </exclusion>
            </exclusions>
        </dependency>  

        <dependency>  
            <groupId>javax.servlet</groupId>  
            <artifactId>javax.servlet-api</artifactId>
            <scope>provided</scope>  
        </dependency>  
        
        <!-- tomcat的支持.-->
		<dependency>
			<groupId>org.apache.tomcat.embed</groupId>
			<artifactId>tomcat-embed-jasper</artifactId>
			<scope>provided</scope>
		</dependency>
```

2.将Jsp页面的前后缀配置加入到application.properties中：
```bash
## JSP配置
# 页面默认前缀
spring.mvc.view.prefix=/WEB-INF/jsp/
# 响应页面默认后缀
spring.mvc.view.suffix=.jsp
```

3.在main下新建webapp文件夹，其下继续新建WEB-INF文件夹，接着创建jsp文件夹，并将list.jsp(展示所有数据信息页面)、add.jsp(增加数据页面)及edit.jsp(编辑数据信息页面)放到jsp文件夹下，项目整体的框架如下：
![](https://raw.githubusercontent.com/butalways1121/img-Blog/master/54.png)
在jsp页面中用到了如下拼接网页相对路径的方法：
```bash
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>
```
其中，request.getContextPath()：用来获取当前的项目根目录路径；request.getSchema()：可以返回当前页面所使用的协议，就是”http”；request.getServerName()返回当前页面所在服务器的名字，就是的“localhost”；request.getServerPort()：返回当前页面所在服务器的端口号，在本例中就是“8088”。例如，在该项目中：`<a href="<%=basePath%>/toEdit"></a>`编辑”中的herf即为：`http://localhost:8088//toEdit`，点击之后就会转到相对应的controller方法。

4.修改controller类的相关方法，实现页面的跳转及对数据的增删改查操作，如下：
```bash
package com.DeptManage.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import com.DeptManage.entity.Dept;
import com.DeptManage.service.DeptService;


@Controller
public class DeptController {

	@Autowired
	private DeptService deptService;
	
	//跳转到list页面，展示所有数据
	@RequestMapping(value = "/list")
	public String list(Model model) {
		System.out.println("查询所有信息！");
		List<Dept> depts=deptService.findAll();
		//model是作为前后台的一个交互作用的,往前台传数据，可以传对象，可以传List，通过el表达式 ${}可以获取到，类似于request.setAttribute("sts",sts)效果一样
        model.addAttribute("depts", depts);
        return "list";
	}
	
	//跳转到edit页面，并将根据id查找到部门数据传入edit
	@RequestMapping("/toEdit")
	 public String toEdit(Model model,Long id) {
        Dept dept = deptService.findById(id);
        model.addAttribute("dept", dept);
        return "edit";
    }

	//编辑部门信息，更新之后再转到list页面
	@RequestMapping("/edit")
    public String edit(Dept dept) {
        deptService.update(dept);
        return "redirect:/list";
    }
	
	//删除部门信息
	@RequestMapping("/toDelete")
    public String delete(Long id) {
        deptService.delete(id);
        System.out.println("删除成功！");
        return "redirect:/list";
    }
	
	//跳转到add页面
	@RequestMapping("/toAdd")
    public String toAdd() {
        return "add";
    }
	
	//添加部门信息，添加完成之后转到list页面
	@RequestMapping("/add")
    public String add(Dept dept) {
        deptService.add(dept);
        System.out.println("添加成功！");
        return "redirect:/list";
    }
}

```
注意这里controller类的注解不能用RestController，因为RestController注解以json的格式返回数据，但是我们有时返回的时候需要跳转界面，所以应该使用Controller这个注解。如果想在某个方法中返回的数据格式是json的话，在该方法上加上ResponseBody这个注解即可。

5.测试
启动App.java，在浏览器中输入`http://localhost:8088/list`，
显示数据库中的所有信息，结果如下：
![](https://raw.githubusercontent.com/butalways1121/img-Blog/master/55.png)
点击删除，则直接将该条数据信息删除；点击编辑或添加则会跳转到相应的页面，输入相关信息提交之后数据库和list页面会同步更新。

***
## 至此，OVER！
