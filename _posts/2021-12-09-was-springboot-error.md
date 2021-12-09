---
layout: post
date:   2020-04-28
categories: [mysql, docker]
title: "基于docker部署MySQL读写分离库"
---


# Error Page in WAS

::Spring offical documentations::

![Image.jpeg]({{site.baseurl}}/images/2021/was-1.jpeg)

::IBM WebSphere how tos add a specific web container custom property.::

   1. Click Servers > Server Types > Application Servers, and select the server for IBM MobileFirst Platform Foundation.
   2. Click Web Container Settings > Web container.
   3. Click Custom properties.
   4. Click New.
   5. Enter the property values listed in the following table.

|             |                                                                                                                      | Table 1. Values for the web container custom property |
| ----------- | -------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| Property    | Value                                                                                                                |                                                       |
| Name        | com.ibm.ws.webcontainer.invokeFlushAfterService                                                                      |                                                       |
| Value       | false                                                                                                                |                                                       |
| Description | See [http://www.ibm.com/support/docview.wss?uid=swg1PM50111](http://www.ibm.com/support/docview.wss?uid=swg1PM50111) |                                                       |

   6. Click OK.
   7. Click Save.

::通过Liberty测试，在WebContainer中设置invokeFlushAfterService="false”的属性，可以跳转404页面。::

