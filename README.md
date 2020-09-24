# ZYC

postman接口自动化测试
背景
日常我们在测试中已经大量使用posman进行接口测试，但是都是基于具体的测试任务，测试完成后脚本就丢弃了，或者是没有很好的复用，没有切实的提升我们日常测试效率。
我们日常对于接口的测试，应该是有如下需求
1.需要快速的构建测试脚本
基于不同的需求，我们需要快速的构建出测试脚本，对具体的需求进行测试，如有新产品，我们需要对这个新产品快速构建出测试脚本，验证产品的功能和流程。
2.需要能够复用，在上线或者日常测试中，实现自动化回归
我们基于第一点的测试脚本，应该有脚本库，在需求测试完成后，将脚本纳入到脚本库中，后续有相关需求变更或者是在全险种测试时，可以执行脚本库中的脚本，达到回归测试的目的。
3.可以快速变更和扩展
我们的测试脚本需要参数化，最好能够是数据驱动的，同一接口，应该只有一套测试脚本，但是可以测试多种场景，不同的场景只有测试数据不同
4.可以读取测试数据，记录测试结果
第三点中的测试数据，可以在多种多样的介质中存储，如txt，excel，db等
5.可以验证数据库数据正确性
对于测试断言，可以是返回值，也可以是数据库数据检查结果
6.可以支持数据传递
即大多数业务场景需要多个接口先后执行，前一个接口的返回数据可以作为后一个接口的入参数据，因此测试工具需要可以方便的存储数据。

基于以上需求，postman大部分满足需求，结合我们目前测试人员的技能水平，可以在自动化起步阶段，使用postman进行整个自动化测试体系的初始化，后续的规划，可以赚到开源管理工具中，实现集中管理。

下面以鼎盛渠道对接为例，进行postman接口自动化测试的介绍
参数化
Postman中，参数化使用 双大括号{{参数名}} 的形式，标记参数，postman中，参数分为几大类
Global   --全局变量
Collection  ---测试集变量
Environment  --环境变量
Data  --数据变量
Local  --本地变量





以上为变量的使用方法
不同的变量作用范围不一致，如果global和local都定义了变量，则以local为准，local的优先级最高



好，接下来看一下投保接口
第一步，参数化

<?xmlversion="1.0" encoding="utf-8"?>
<packageList>
<package>
<header>
<requestType>insure</requestType>
<requestId>T0685TY-{{requestId}}</requestId>
</header>
<request>
<insurePlan>
<productCode>{{productCode}}</productCode>
<proposalPlan>{{proposalPlan}}</proposalPlan>
<operateTime>{{now}}</operateTime>
<effectTime>{{effectTime}}</effectTime>
<expiryTime>{{expiryTime}}</expiryTime>
<sumPremium>{{premium}}</sumPremium>
<journey>北京天安门</journey>
<yearFlag>{{yearFlag}}</yearFlag>
</insurePlan>
<application>
<name>{{application.name}}</name>
<gender>106001</gender>
<birthday>{{application.birthday}}</birthday>
<idType>{{application.idType}}</idType>
<idNo>{{application.idNo}}</idNo>
<mobile>15077862813</mobile>
<email>zhanghailiang@sinosoft.com.cn</email>
<address>jiangsunanjing</address>
</application>
<insureds>
<insured>
<seqNo>1</seqNo>
<name>{{insured.name}}</name>
<eName>heyden</eName>
<gender>106001</gender>
<birthday>{{insured.birthday}}</birthday>
<idType>{{insured.idType}}</idType>
<idNo>{{application.idNo}}</idNo>
<mobile>15077862813</mobile>
<address>{{insured.adress}}</address>
<email>zhanghailiang@sinosoft.com.cn</email>
<benifyType>644006001</benifyType>
<countryCode>CN</countryCode>
<relationShip>601005</relationShip>
</insured>
</insureds>
</request>
</package>
</packageList>

可以看到，我们将绝大部分关键字段参数化（可以根据需要，将全部字段参数化）
第二步，设置参数值
在pre-requestScript中设置参数值
Pre-requestScript是在请求发送前，执行的脚本，使用的是java script

从第一步中，我们设置的参数，包括操作时间，起保日期，保险止期，投保人，被保险人，投保人证件号，被保险人证件号码，证件类型，整年标志等
这里面要解决几个问题。
1.如何获取当前时间，因为我们再测试时，不可能每次修改时间参数，最方便的做法是获取系统当前时间
2.生效时间和保险止期，生效时间需要是当前时间后10分钟之后，保险止期需要根据保险期间自动计算
3.每个请求的订单号，投保人被保险人证件号不能重复。

解决完以上问题后，可以将设置好的值，赋给第一步的变量

1.日期

// 处理保险期限，因为鼎盛的保险期限都是当天生效，所以我们传入的保险期限参数减1天
var days = pm.variables.replaceIn('{{insurancedate}}') - 1;
// 控制台输出days值，看一下是否正确
console.log(days);

//处理当前时间，以下获取当前时间的基本逻辑是，new一个date对象，分别获取年，月，日，小时，分钟和秒，当月，日，小时，分钟和秒小于10的时候，前面加0.
// 同时也处理了当前时间10分钟后起保的问题
var myDate=new Date();
var date = myDate.getDate();
var year = myDate.getFullYear();
// 获取的月份是从0开始，因此+1得到当前月份
var month = myDate.getMonth() + 1;
var datetody = myDate.getDate();
var hours = myDate.getHours() ;
// 获取的分钟数+20，做为保险起期
var min = myDate.getMinutes() + 20;
// 加10分钟才是当前分钟数，具体为什么不知道
var mincurrent = myDate.getMinutes()+10; 
var seconds = myDate.getSeconds();

datetody = datetody < 10? "0"+ datetody : datetody;
date = date < 10? "0"+ date : date;
month = month <10 ? "0" + month : month;
hours = hours < 10 ? "0" + hours : hours;
min = min < 10 ? "0" + min : min;
mincurrent = mincurrent < 10 ? "0" + mincurrent : mincurrent
seconds = seconds < 10 ? "0" + seconds : seconds;

// 处理保险止期的处理逻辑是 先获取当前时间，然后把当前时间加上保险期限，保险期限是后面会从数据文件中传进来，先把保险期限计算成毫秒，然后与当前时间相加得到 ms的值，最后new Date(ms),获取相加后的日期
currentdate = new Date();
var ms = currentdate.getTime();
//保险止期 计算为天转化为毫秒
ms+=days*24*60*60*1000;
//得到保险止期时间
var expirydate = new Date(ms);
//得到保险止期的年月日
var expiryyear = expirydate.getFullYear();
var expiryday = expirydate.getDate();
var expirymonth = expirydate.getMonth() + 1;

expiryday = expiryday < 10? "0"+ expiryday : expiryday;
expirymonth = expirymonth <10 ? "0" + expirymonth : expirymonth;

//操作时间，即当前时间
var operateDate = year + "-" + month +"-" + datetody +" "+ hours +":" + mincurrent +":"+ seconds;
//生效时间，加了10分钟后的时间，赋值给effectTime
pm.collectionVariables.set("effectTime", year + "-" + month +"-" + datetody +" "+ hours +":" + min +":"+ seconds );
//到期时间，默认到期时间为1天,止期为23点59分59秒 expiryTime
pm.collectionVariables.set("expiryTime", expiryyear + "-" + expirymonth +"-" + expiryday +" "+ "23:59:59" );
// console.log(expiryTime)
//赋值给当前时间now
pm.globals.set("now", year + "-" + month +"-" + datetody +" "+ hours +":" + mincurrent +":"+ seconds );


以上，便是处理了处理了第一个和第二个问题，日期和分钟数的问题
第三个问题，即订单和证件号码的不能重复的问题
Postman提供了随机数据的功能，即{{$}}开头的参数，具体可以参照官方文档
https://learning.postman.com/docs/writing-scripts/script-references/variables-list/

我们使用了1到1000的随机整数功能
$randomInt
//requestID部分，使用了3个连续随机数，因为只有一个随机数的时候，重复度较高，3个连续的随机数可以提供1000*1000*1000个随机数，对于日常测试够用了
pm.collectionVariables.set("requestId", "068520" + pm.variables.replaceIn('{{$randomInt}}') + pm.variables.replaceIn('{{$randomInt}}') + pm.variables.replaceIn('{{$randomInt}}'));


注意，对于pre-requestScript中，使用参数时，需要使用pm.variables.replaceIn方法，不能直接使用{{}}来引用参数

整体的参数值设置代码如下
// 给默认值，1
pm.collectionVariables.set("insurancedate", 1);
// 处理保险期限
var days = pm.variables.replaceIn('{{insurancedate}}') - 1;
console.log(days);

//处理当前时间
var myDate=new Date();
var date = myDate.getDate();
var year = myDate.getFullYear();
var month = myDate.getMonth() + 1;
var datetody = myDate.getDate();
var hours = myDate.getHours() ;
var min = myDate.getMinutes() + 20;
var mincurrent = myDate.getMinutes()+10;
var seconds = myDate.getSeconds();

datetody = datetody < 10? "0"+ datetody : datetody;
date = date < 10? "0"+ date : date;
month = month <10 ? "0" + month : month;
hours = hours < 10 ? "0" + hours : hours;
min = min < 10 ? "0" + min : min;
mincurrent = mincurrent < 10 ? "0" + mincurrent : mincurrent
seconds = seconds < 10 ? "0" + seconds : seconds;

// 处理保险止期
currentdate = new Date();
var ms = currentdate.getTime();
//保险止期 计算为天转化为毫秒
ms+=days*24*60*60*1000;
//得到保险止期时间
var expirydate = new Date(ms);
//得到保险止期的年月日
var expiryyear = expirydate.getFullYear();
var expiryday = expirydate.getDate();
var expirymonth = expirydate.getMonth() + 1;

expiryday = expiryday < 10? "0"+ expiryday : expiryday;
expirymonth = expirymonth <10 ? "0" + expirymonth : expirymonth;

//操作时间，即当前时间
var operateDate = year + "-" + month +"-" + datetody +" "+ hours +":" + mincurrent +":"+ seconds;
//生效时间
pm.collectionVariables.set("effectTime", year + "-" + month +"-" + datetody +" "+ hours +":" + min +":"+ seconds );
//到期时间，默认到期时间为1天,止期为23点59分59秒
pm.collectionVariables.set("expiryTime", expiryyear + "-" + expirymonth +"-" + expiryday +" "+ "23:59:59" );
// console.log(expiryTime)
//赋值给当前时间
pm.globals.set("now", year + "-" + month +"-" + datetody +" "+ hours +":" + mincurrent +":"+ seconds );
//默认的请求id，随机的uuid-v4 style
//如 611c2e81-2ccb-42d8-9ddc-2d0bfa65c1b4
// pm.collectionVariables.set("requestId", "{{$guid}}" );

pm.collectionVariables.set("requestId", "068520" + pm.variables.replaceIn('{{$randomInt}}') + pm.variables.replaceIn('{{$randomInt}}') + pm.variables.replaceIn('{{$randomInt}}'));
//产品代码  06851F
//户外休闲     06851F
pm.collectionVariables.set("productCode", "06851F" );
//计划代码
// 计划一 644001001
// 计划二 644001002
// 计划三 644001003
pm.collectionVariables.set("proposalPlan", "644001001" );
// now
// effectTime
// expiryTime
// 保费，精确到分，由数据提供，
// premium
pm.collectionVariables.set("premium", "300" );
// yearFlag，整年投保标识
pm.collectionVariables.set("yearFlag", "0" );
// 投保人名称
// application.name
pm.collectionVariables.set("application.name", "张三" );
// 生日
// application.birthday
pm.collectionVariables.set("application.birthday", "1982-09-09" );
// 证件类型
// application.idType
// 个人：
// 120001 身份证
// 120002 军官证
// 120004 台胞证
// 120005 护照
// 120006 港澳返乡证
// 法人：
// 110004 统一社会信用
// 证代码
// 110003 机构组织代码
pm.collectionVariables.set("application.idType", "120002" );
// 证件id
// application.idNo
var idNo = "2020082"+pm.variables.replaceIn('{{$randomInt}}')+pm.variables.replaceIn('{{$randomInt}}')+pm.variables.replaceIn('{{$randomInt}}');
pm.collectionVariables.set("application.idNo", idNo  );
//insured.name
// 被保险人姓名
pm.collectionVariables.set("insured.name", "张三" );
// 被保险人生日
// insured.birthday
pm.collectionVariables.set("insured.birthday", "1982-09-09" );
// insured.idType
// 被保险人类型
// 个人：
// 120001 身份证
// 120002 军官证
// 120004 台胞证
// 120005 护照
// 120006 港澳返乡证
// 120007 出生证明
pm.collectionVariables.set("insured.idType", "120002" );
// insured.idNO
// 被保险人证件号码
pm.collectionVariables.set("insured.idNo", idNo );
// 被保险人履行地址
// insured.adress
pm.collectionVariables.set("insured.adress", "南京" );


测试
脚本和参数都设置好以后，就是要对返回结果进行断言以及保存返回参数了
Postman中，测试是在页面的test 页中


Test是在请求发送后，服务器返回结果后执行的脚本，也是js编写
// 以下是断言返回码是200，并且返回的xml中，字段errorMessage的值是“成功”
// 大体逻辑是 判断返回的code，等于200，将返回的xml解析成json格式，并判断errorMessage的值为成功
pm.test("返回成功", () => {
  pm.expect(pm.response.code).to.eql(200);
  const responseJson = xml2Json(pm.response.text());
  pm.expect(responseJson.packageList.package.response.errorMessage).to.eql('成功');
});

服务器返回的xml如下

<?xml version="1.0" encoding="UTF-8"?>
<packageList><package><header><requestType>insure</requestType><sendTime>2020-08-24 22: 06: 53</sendTime><requestId>T0685TY-06852029982183</requestId></header><response><policyNo>E55010106642020A009069</policyNo><downloadPolicyUrl></downloadPolicyUrl><responseCode>0000</responseCode><errorMessage>成功</errorMessage></response></package></packageList>

断言的语法，可以在postman编写脚本时查看，如下图
我们常用的的有
//判断返回码是200
pm.expect(pm.response.code).to.eql(200);
// 判断返回内容等于 .eql(‘’)
pm.expect(responseJson.packageList.package.response.errorMessage).to.eql('成功')
// 判断返回内容包含
pm.expect(responseJson.packageList.package.response.errorMessage).to.include("注销成功");



实际执行时，结果可以看到测试的结果是通过还是失败




参数文件
前面是设置了脚本的参数化，那么如何根据测试数据批量执行呢？
点击posman页面的runner

选中要执行的测试集

选中测试数据

预览测试数据

可以看到测试数据的内容
测试数据的制作注意一点
第一行标题内容，必须与参数名相同，postman会自动进行匹配，如果某个变量没有在数据文件中配置，则取默认值（在上一章节脚本中设置的）
数据文件的参数优先级要高于collection的优先级。
参数文件内容如下

Postman会逐行读取参数，并执行，如下为执行结果


常见问题
1.纯数字字符串读取的问题
当使用csv存储数据时，纯数字的参数，尤其是以0开头的数字，excel会转化为数字格式，并把开头的0去掉，此时，需要使用txt打开csv文件，将数字用双引号””包裹，注意，后面一旦再用excel打开并保存时，excel还会将数据再转化为数字（暂未找到其他解决办法），或者用txt存储测试数据
2.测试中日志的查看
可以通过console查看数据，包括发送的和返回的都可以
在单接口测试执行时，可以通过左下角的console查看

在runner批量执行时，可以通过view-show postman console查看

3.测试用例管理
Postman的team功能，暂未使用过，可以共享用例和脚本（可能有商业限制），postman的脚本共享和传递是一个问题，可以暂时先将自动化测试资产积累起来，后续迁移到开源自动化管理平台中
