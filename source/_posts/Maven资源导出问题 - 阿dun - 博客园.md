---
title: Maven资源导出问题
date: 2021-03-05 16:55
categories: ['Java']
tags: ['Maven', 'Java']
---
**Maven由于它的约定大于配置，可能在开发过程中遇到我们写的配置文件，无法被导出或者生效的问题，尝试在Maven中加上如下配置。**

    
    
     <!--在build中配置resources，来防止我们资源导出失败的问题-->
    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
    

记录一下方便自己查看

