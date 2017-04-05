---
layout: post
title:  "JSP、Servlet中的路径问题"
subtitle:   " \"Hello World, Hello Blog1\""
author:     "李航"
date:   2017-01-04 23:35:59 +0800
catalog: true
tags:	Java Servlet Jsp
---
##JSP、Servlet中的路径问题
###1.JSP中获得当前应用的相对路径和绝对路径

#####当前我们访问classroom项目下的login.jsp页面

>得到项目名：request.getContextPath();

/classroom

>得到当前页面所在目录下全名称：request.getServletPath()

/pages/login.jsp

>得到包含工程名的当前页面全路径:request.getRequestURI();

/classroom/pages/login.jsp

>项目的绝对路径：application.getRealPath();

F:/apache-tomcat-8.0.32/wtpwebapps/Classroom

>得到页面所在服务器的绝对路径：

>new File(application.getRealPath(request.getServletPath())).getParent()；

F:/apache-tomcat-8.0.32/wtpwebapps/Classroom/pages/login.jsp

###2.Servlet中获得当前应用的相对路径和绝对路径

>得到请求页面的url：request.getRequestURL();

http://localhost:8080/classroom/pages/login.jsp

>得到请求页面的uri（说是得到包含工程名的当前页面全路径：
request.getRequestURI();

/classroom/pages/login.jsp

>得到项目名：request.getContextPath()

/classroom;

>得到请求的页面名：request.getServletPath() 

/login.jsp

>得项目所在服务器的绝对路径：request.getSession().getServletContext().getRealPath("")  

F:/apache-tomcat-8.0.32/wtpwebapps/Classroom

###3.JSP、Servlet中的相对路径

a) 在JSP中

“/”代表站点（服务器）根目录http://127.0.0.1/

b) 在Servlet中

“/”代表Web应用的根目录http://127.0.0.1/JSP_Servlet_Path/

request.getRequestDispatcher("/a/a.jsp").forward(request, response);

相对路径/a/a.jsp中/相对于web应用的根目录，所以相当于请求跳转到绝对路径

http://127.0.0.1/JSP_Servlet_Path/a/a.jsp

response.sendRedirect("/JSP_Servlet_Path/b/b.jsp");

因为重定向中的方法是传递给浏览器，用于重新发送请求的，而在浏览器端“/”代表，相对于站点根目录http://127.0.0.1/，所以这里必须要加上/JSP_Servlet_Path，这样浏览器重新请求的地址为：http://127.0.0.1/JSP_Servlet_Path/b/b.jsp







