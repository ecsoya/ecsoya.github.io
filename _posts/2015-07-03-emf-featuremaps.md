---
title: "EMF FeatureMap"
layout: post
date:   2015-07-02
categories: [eclipse]
---

### 前言

FeatureMap是EMF中的一个非常好的功能，一方面它能用来深度解析XML Schema（Wildcard等），另一方面，如果对XML Schema不是太熟悉的人，也可以用它来定义模型。

首先，FeatureMap虽然名字带着Map，但它其实是一个List，`public interface FeatureMap extends EList<FeatureMap.Entry>`。

其次，FeatureMap.Entry跟Map.Entry类似，key值是EStructuredFeature，value值对应一个具体的值。也就是说一个FeatureMap中可以映射某个EObject对象的多个feature的不同的值。

有了这些概念，下面就来介绍一些具体的用法。

### Group的实现方法

首先我们还是看这个经典的例子：

![FeatureMap - 1 ]({{site.baseurl}}/images/2015-07-03-featuremap-1.gif)

> Supplier中包含preferredOrders和standardOrders。
>
> preferredOrders和standardOrders的类型都是PurchaseOrder。

另外我们需要统计所有的orders，如下图：

![FeatureMap - 2 ]({{site.baseurl}}/images/2015-07-03-featuremap-2.gif)

> Supplier中所有的preferredOrders和standardOrders都应该被包含在orders中。

所以，我们有了最终的模型：

![FeatureMap - 3 ]({{site.baseurl}}/images/2015-07-03-featuremap-3.gif)

有了模型，我们就要想怎么去在Java代码中实现，可能你也想好了，这样简单的逻辑，有n中实现方式，何苦呢？

我们今天的主题是FeatureMap，所以我们在这讨论的肯定不是写Java代码去实现。

#### FeatureMap.group的实现方法

* 第一步：定义EClass Supplier和PurchaseOrder以及EEnum OrderKind。

* 第二步：在Supplier中，定义一个*EAttribute*，命名为orders，选择类型为*EFeatureMapEntry*，并将Upper Bounds改为-1备用。

* 第三步：在Supplier中，定义两个*EReference*，分别命名为preferredOrders和standardOrders，选择类型为PurchaseOrder，并将Upper Bounds改为-1备用。

* 第四步：给EAttribute orders添加一个EAnnotation[extendedMetaData="kind='group'"]。

![FeatureMap - 4 ]({{site.baseurl}}/images/2015-07-03-featuremap-4.gif)   

* 第五步：分别给EReference preferredOrders和standardOrders也添加一个EAnnotation[extendedMetaData="group='#orders'"]。

![FeatureMap - 5 ]({{site.baseurl}}/images/2015-07-03-featuremap-5.gif)）

* 第六步：分别将EReference preferredOrders和standardOrders的Transient和Volatile属性设置为true。

![FeatureMap - 6 ]({{site.baseurl}}/images/2015-07-03-featuremap-6.gif)）

好了，可以生成你的Java代码了：

----------------------------
	public FeatureMap getOrders() {
		if (orders == null) {
			orders = new BasicFeatureMap(this, OrdersPackage.SUPPLIER__ORDERS);
		}
		return orders;
	}
	
	public EList<PurchaseOrder> getPreferredOrders() {
		return getOrders().list(OrdersPackage.Literals.SUPPLIER__PREFERRED_ORDERS);
	}

	public EList<PurchaseOrder> getStandardOrders() {
		return getOrders().list(OrdersPackage.Literals.SUPPLIER__STANDARD_ORDERS);
	}
----------------------------
>
> #### 需要注意的地方:
>   1. 用来实现group的属性一定要定义为EAttribute，并且类型为EFeatureMapEntry而不是EFeatureMap。
>   2. 定义EAnnotation的时候要注意ExtendedMetaData的URI为：http:///org/eclipse/emf/ecore/util/ExtendedMetaData
>   3. 定义组合属性的EAnnotation的时候，只能定义[key="kind", value="group"]。 
>   4. 定义分组属性的EAnnotation的时候，只能定义[key="group", value="#组合属性名"]。
>   5. 一定要将分组属性的Transient和Volatile属性设置为true，否则会按照EMF默认的规则生成代码。
>
