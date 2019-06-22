# 30_XML
[TOC]
随着互联网的发展，Web应用程序的丰富，开发人员越来越希望能够使用客户端来操作XML技术。而XML技术一度成为存储和传输结构化数据的标准。所以，本章就详细探讨一下JavaScript中使用XML的技术。
对于什么是XML，干什么用的，这里就不在赘述了，在以往的XHTML或PHP课程都有涉及到，可以理解成一个微型的结构化的数据库，保存一些小型数据用的。

## 一．IE中的XML
在统一的正式规范出来以前，浏览器对于XML的解决方案各不相同。DOM2级提出了动态创建XML DOM规范，DOM3进一步增强了XML DOM。所以，在不同的浏览器实现XML的处理是一件比较麻烦的事情。

1.创建XMLDOM对象
IE浏览器是第一个原生支持XML的浏览器，而它是通过ActiveX对象实现的。这个对象，只有IE有，一般是IE9之前采用。微软当年为了开发人员方便的处理XML，创建了MSXML库，但却没有让Web开发人员通过浏览器访问相同的对象。

var xmlDom = new ActiveXObject('MSXML2.DOMDocument');

ActiveXObject类型
XML版本字符串	说明
Microsoft.XmlDom	最初随同IE发布，不建议使用
MSXML2.DOMDocument	脚本处理而更新的版本，仅在特殊情况作为备份用
MSXML2.DOMDocument.3.0	在JavaScript中使用，这是最低的建议版本
MSXML2.DOMDocument.4.0	脚本处理时并不可靠，使用这个版本导致安全警告
MSXML2.DOMDocument.5.0	脚本处理时并不可靠，使用这个版本导致安全警告
MSXML2.DOMDocument.6.0	脚本能够可靠处理的最新版本

PS：在这六个版本中微软只推荐三种：
1.MSXML2.DOMDocument.6.0 最可靠最新的版本
2.MSXML2.DOMDocument.3.0 兼容性较好的版本
3.MSXML2.DOMDocument    仅针对IE5.5之前的版本

PS：这三个版本在不同的windows平台和浏览器下会有不同的支持，那么为了实现兼容，我们应该考虑这样操作：从6.0->3.0->备用版本这条路线进行实现。
function createXMLDOM() {
	var version = [
							'MSXML2.DOMDocument.6.0',
							'MSXML2.DOMDocument.3.0',
							'MSXML2.DOMDocument'
	];
	for (var i = 0; i < version.length; i ++) {
		try {
			var xmlDom = new ActiveXObject(version[i]);
			return xmlDom;
		} catch (e) {
			//跳过
		}
	}
	throw new Error('您的系统或浏览器不支持MSXML！');		//循环后抛出错误
}

2.载入XML
如果已经获取了XMLDOM对象，那么可以使用loadXML()和load()这两个方法可以分别载入XML字符串或XML文件。
xmlDom.loadXML('<root version="1.0"><user>Lee</user></root>');
alert(xmlDom.xml);

PS：loadXML参数直接就是XML字符串，如果想效果更好，可以添加换行符\n。.xml属性可以序列化XML，获取整个XML字符串。
xmlDom.load('test.xml');					//载入一个XML文件
alert(xmlDom.xml);

当你已经可以加载了XML，那么你就可以用之前学习的DOM来获取XML数据，比如标签内的某个文本。
	var user = xmlDom.getElementsByTagName('user')[0];	//获取<user>节点
	alert(user.tagName);								//获取<user>元素标签
	alert(user.firstChild.nodeValue);						//获取<user>里的值Lee

DOM不单单可以获取XML节点，也可以创建。
var email= xmlDom.createElement('email');
xmlDom.documentElement.appendChild(email);

3.同步及异步
load()方法是用于服务器端载入XML的，并且限制在同一台服务器上的XML文件。那么在载入的时候有两种模式：同步和异步。
所谓同步：就是在加载XML完成之前，代码不会继续执行，直到完全加载了XML再返回。好处就是简单方便、坏处就是如果加载的数据停止响应或延迟太久，浏览器会一直堵塞从而造成假死状态。
xmlDom.async = false;						//设置同步，false，可以用PHP测试假死

所谓异步：就是在加载XML时，JavaScript会把任务丢给浏览器内部后台去处理，不会造成堵塞，但要配合readystatechange事件使用，所以，通常我们都使用异步方式。
xmlDom.async = true;						//设置异步，默认

通过异步加载，我们发现获取不到XML的信息。原因是，它并没有完全加载XML就返回了，也就是说，在浏览器内部加载一点，返回一点，加载一点，返回一点。这个时候，我们需要判断是否完全加载，并且可以使用了，再进行获取输出。

XML DOM中readystatechange事件
就绪状态	说明
1	DOM正在加载
2	DOM已经加载完数据
3	DOM已经可以使用，但某些部分还无法访问
4	DOM已经完全可以
PS：readyState可以获取就绪状态值

var xmlDom = createXMLDOM();
xmlDom.async = true;						//异步，可以不写
xmlDom.onreadystatechange = function () {		
	if (xmlDom.readyState == 4) {			//完全加载了，再去获取XML
		alert(xmlDom.xml);
	}
}
xmlDom.load('test.xml');					//放在后面重点体现异步的作用

PS：可以通过readyState来了解事件的执行次数，将load()方法放到最后不会因为代码的顺序而导致没有加载。并且load()方法必须放在onreadystatechange之后，才能保证就绪状态变化时调用该事件处理程序，因为要先触发。用PHP来测试，在浏览器内部执行时，是否能操作，是否会假死。

PS：不能够使用this，不能够用IE的事件处理函数，原因是ActiveX控件为了预防安全性问题。

PS：虽然可以通过XML DOM文档加载XML文件，但公认的还是XMLHttpRequest对象比较好。这方面内容，我们在Ajax章节详细了解。

4.解析错误
在加载XML时，无论使用loadXML()或load()方法，都有可能遇到XML格式不正确的情况。为了解决这个问题，微软的XML DOM提供了parseError属性。

parseError属性对象
属性	说明
errorCode	发生的错误类型的数字代号
filepos	发生错误文件中的位置
line	错误行号
linepos	遇到错误行号那一行上的字符的位置
reason	错误的解释信息

	if (xmlDom.parseError == 0) {
		alert(xmlDom.xml);
	} else {
		throw new Error('错误行号：' + xmlDom.parseError.line +
					'\n错误代号：' + xmlDom.parseError.errorCode + 
					'\n错误解释：' + xmlDom.parseError.reason);
	}

## 二．DOM2中的XML
IE可以实现了对XML字符串或XML文件的读取，其他浏览器也各自实现了对XML处理功能。DOM2级在document.implementaion中引入了createDocument()方法。IE9、Firefox、Opera、Chrome和Safari都支持这个方法。

1.创建XMLDOM对象
var xmlDom = document.implementation.createDocument('','root',null);	//创建xmlDom
var user = xmlDom.createElement('user');							//创建user元素
xmlDom.getElementsByTagName('root')[0].appendChild(user);			//添加到root下
var value = xmlDom.createTextNode('Lee');							//创建文本
xmlDom.getElementsByTagName('user')[0].appendChild(value);		//添加到user下
alert(xmlDom.getElementsByTagName('root')[0].tagName);
alert(xmlDom.getElementsByTagName('user')[0].tagName);
alert(xmlDom.getElementsByTagName('user')[0].firstChild.nodeValue);

PS：由于DOM2中不支持loadXML()方法，所以，无法简易的直接创建XML字符串。所以，只能采用以上的做法。
PS：createDocument()方法需要传递三个参数，命名空间，根标签名和文档声明，由于JavaScript管理命名空间比较困难，所以留空即可。文档声明一般根本用不到，直接null即可。命名空间和文档声明留空，表示创建XMLDOM对象不需要命名空间和文档声明。
PS：命名空间的用途是防止太多的重名而进行的分类，文档类型表明此文档符合哪种规范，而这里创建XMLDOM不需要使用这两个参数，所以留空即可。
2.载入XML
DOM2只支持load()方法，载入一个同一台服务器的外部XML文件。当然，DOM2也有async属性，来表面同步或异步，默认异步。 
//同步情况下
var xmlDom = document.implementation.createDocument('','root',null);
xmlDom.async = false;
xmlDom.load('test.xml');
alert(xmlDom.getElementsByTagName('user')[0].tagName);

//异步情况下
var xmlDom = document.implementation.createDocument('','root',null);
xmlDom.async = true;
addEvent(xmlDom, 'load', function () {					//异步直接用onload即可
	alert(this.getElementsByTagName('user')[0].tagName);
});
xmlDom.load('test.xml');

PS：不管在同步或异步来获取load()方法只有Mozilla的Firefox才能支持，只不过新版的Opera也是支持的，其他浏览器则不支持。

3.DOMParser类型
由于DOM2没有loadXML()方法直接解析XML字符串，所以提供了DOMParser类型来创建XML DOM对象。IE9、Safari、Chrome和Opera都支持这个类型。
var xmlParser = new DOMParser();						//创建DOMParser对象
var xmlStr = '<user>Lee</user></root>';	//XML字符串
var xmlDom = xmlParser.parseFromString(xmlStr, 'text/xml');	//创建XML DOM对象
alert(xmlDom.getElementsByTagName('user')[0].tagName);	//获取user元素标签名

PS：XML DOM对象是通过DOMParser对象中的parseFromString方法来创建的，两个参数：XML字符串和内容类型text/xml。

4.XMLSerializer类型
由于DOM2没有序列化XML的属性，所以提供了XMLSerializer类型来帮助序列化XML字符串。IE9、Safari、Chrome和Opera都支持这个类型。
var serializer = new XMLSerializer();			//创建XMLSerializer对象
var xml = serializer.serializeToString(xmlDom);	//序列化XML
alert(xml);

5.解析错误
在DOM2级处理XML发生错误时，并没有提供特有的对象来捕获错误，而是直接生成另一个错误的XML文档，通过这个文档可以获取错误信息。
var errors = xmlDom.getElementsByTagName('parsererror');
if (errors.length > 0) {
	throw new Error('XML格式有误：' + errors[0].textContent);
}

PS：errors[0].firstChild.nodeValue也可以使用errors[0].textContent来代替。

三．跨浏览器处理XML
如果要实现跨浏览器就要思考几个个问题：1.load()只有IE、Firefox、Opera支持，所以无法跨浏览器；2.获取XML DOM对象顺序问题，先判断先进的DOM2的，然后再去判断落后的IE；3.针对不同的IE和DOM2级要使用不同的序列化。4.针对不同的报错进行不同的报错机制。

//首先，我们需要跨浏览器获取XML DOM
function getXMLDOM(xmlStr) {
	var xmlDom = null;
	
	if (typeof window.DOMParser != 'undefined') {		//W3C
		xmlDom = (new DOMParser()).parseFromString(xmlStr, 'text/xml');
		var errors = xmlDom.getElementsByTagName('parsererror');
		if (errors.length > 0) {
			throw new Error('XML解析错误：' + errors[0].firstChild.nodeValue);
		}
	} else if (typeof window.ActiveXObject != 'undefined') {	//IE
		var version = [
						'MSXML2.DOMDocument.6.0',
						'MSXML2.DOMDocument.3.0',
						'MSXML2.DOMDocument'
		];
		for (var i = 0; i < version.length; i ++) {
			try {
				xmlDom = new ActiveXObject(version[i]);
			} catch (e) {
				//跳过
			}
		}
		xmlDom.loadXML(xmlStr);
		if (xmlDom.parseError != 0) {
			throw new Error('XML解析错误：' + xmlDom.parseError.reason);
		}
	} else {
		throw new Error('您所使用的系统或浏览器不支持XML DOM！');
	}
	
	return xmlDom;
}

//其次，我们还必须跨浏览器序列化XML
function serializeXML(xmlDom) {
	var xml = '';
	if (typeof XMLSerializer != 'undefined') {
		xml = (new XMLSerializer()).serializeToString(xmlDom);
	} else if (typeof xmlDom.xml != 'undefined') {
		xml = xmlDom.xml;
	} else {
		throw new Error('无法解析XML！');
	}
	return xml;
}

PS：由于兼容性序列化过程有一定的差异，可能返回的结果字符串可能会有一些不同。至于load()加载XML文件则因为只有部分浏览器支持而无法跨浏览器。

