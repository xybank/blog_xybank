---
title: ewp中模拟报文的使用
date: 2018-06-03 21:46:55
tags: ewp中模拟报文的使用
categories: 方振养
typora-root-url: ..
---

​	**ewp项目中开发页面时是不是有遇到以下问题：**

- 项目中想要模拟各种业务场景是不是很麻烦

- 接口有时候会报错导致无法进行联调

- ...

  现在我给大家提供一个小工具，可以解决上面说的一些问题，有助于提高开发效率。

<!--more-->

#### 一、代码预览

1. ##### 配制文件部分

   ```
   {cib_conf, [
   	{root, "/var/www/apps/fangzy/script/xml"},%根路径
   	{mode, "cib_fwgl_new"},%模块
   	{errorpage, "errorpage.xml"},%app动户签约报错页面
   	{form50000_1, "form50000_1.xml"},%app动户签约查询页面
   	{form50000_2, "form50000_2.xml"},%app动户签约签约页面
   	{form50000_4, "form50000_4.xml"}%app动户签约修改页面
   	]}.
   ```

2. ##### 代码部分

   ```
   -module('fangzy').
   -include("ewp.hrl").
   -compile([export_all]).

   get_mobileNum() ->
       case ?param("phoneNum","") of
         "" ->
             {_, Session} = session_service:read(),
             case  proplists:get_value(xydffmobile,Session) of
                 undefined -> [];
                 Mobile -> Mobile
             end;
         PhoneNo ->
             PhoneNo
       end.

   spacial_char_filter(Res) ->
      spacial_char_filter(Res, xml).

   spacial_char_filter(Res, RetType) when is_binary(Res)->
       spacial_char_filter(binary_to_list(Res), RetType);
   spacial_char_filter(Res, json) when is_list(Res)->
       spacial_char_filter1(Res);
   spacial_char_filter(Res, xml) when is_list(Res)->
       Index1 = string:str(Res,"<hsmbs>"),
       Str_len = string:len(Res),
       Res1 = string:substr(Res, Index1, Str_len),
       Res2 = spacial_char_filter1(Res1),
       Res3 = re:replace(Res2,"\"","“",[global,{return, list}]),
       Res4 = re:replace(Res3,"\\{","",[global,{return, list}]),
       Res5 = re:replace(Res4,"\\}","",[global,{return, list}]),
       Res6 = re:replace(Res5,"\\[","【",[global,{return, list}]),
       Res7 = re:replace(Res6,"\\]","】",[global,{return, list}]),
       "<?xml version=\"1.0\"  encoding=\"utf-8\"?>" ++ Res7.

   %xml报文格式化
   xmlFormate() ->
       ResBody = 
           case ResType of
               "json" ->
                   Res_json1 = cib_utils:spacial_char_filter(Res1),
                   spacial_char_filter(Res_json1, json);
               _ ->
                   Res2 = cib_utils:spacial_char_filter(Res1),
                   Res = spacial_char_filter(Res2),
                   Doc = xml_eng:xml_to_term(Res),
                   case ?ewp_xpath("hsmbs/tranCode", Doc) of
                       ["form00002_3"] ->
                           case get_mobileNum() of
                               [] ->
                                   % update_jsession(?ewp_xpath("/hsmbs/jsessionid",Doc), AdapterName),
                                   Res;
                               Mobile ->
                                   Req_type = ?param("req_type", "card"),
                                   [_, _, _, DeviceId] = login:get_mes(),
                                   % update_jsession(?ewp_xpath("/hsmbs/jsessionid",Doc), AdapterName),

                                   Res3 = ebank_utils:invoke_procedure("simulator","app", [{mobile, Mobile}, {cli_dev, DeviceId},{req_type, Req_type}], [], [{url, "00002.cli?flowsn=6664"}]),
                                   [Cardtype] = ?ewp_xpath("hsmbs/cardtype", Doc),
                                   Index1 = string:str(Res3,"</hsmbs>"),
                                   Res4 = string:substr(Res3, 1, Index1-1),
                                   Res4 ++ "<cardtype>"++ Cardtype ++"</cardtype></hsmbs>"
                               end;
                       ["form10071"] ->
                           %%取出卡列表数据并排序
                           % Cardlist = xml_eng:xpath_term_value("hsmbs/body/cardlist", Doc),
                           Dyjr="3",
                           Form10071_1_url = home:get_dyjr_url(Dyjr),
                         

                           "<?xml version=\"1.0\" encoding=\"utf-8\"?><hsmbs><url>"++ Form10071_1_url ++"</url><tranCode>form10071</tranCode></hsmbs>";
                       ["form99000"] ->
                           Dyjr="2",
                           Dyjr_Url = home:get_dyjr_url(Dyjr),
                           "<?xml version=\"1.0\" encoding=\"utf-8\"?><hsmbs><url>"++ Dyjr_Url ++"</url><openTimer>true</openTimer><tranCode>form99000</tranCode></hsmbs>";
                       ["form50500"] ->
                                   Group2 = ?ewp_xpath("hsmbs/body/relacc/group", Doc),
                                   Cardindex = ?ewp_xpath("hsmbs/body/cardindex/value", Doc),
                                   GoTypeValue = ?ewp_xpath("hsmbs/body/type/value", Doc),
                                   Listno = lists:map(fun ({_,Group3}) ->
                                       Acc2 = proplists:get_value(acc,Group3),
                                       Aname = proplists:get_value(aname,Group3),
                                       CardList = "\{\"cardNo\":" ++ "\"" ++ Acc2 ++ "\",\"alias\":"++ "\"" ++ Aname ++"\"\},",
                                       CardList
                                   end,Group2),
                                   Listno_val = "\{\"data\":[" ++ Listno ++ "]\}",
                                   Listno_val2 = re:replace(Listno_val,",]\}","]\}",[{return,list},global]),
                                   Xykcx = cib_controller:new_xykcx(Listno_val2,"xykcx"),
                                   Xykcx_url = Xykcx ++ "&goTypeValue=" ++ GoTypeValue ++ "&accountIndex=" ++ Cardindex,
                                   "<?xml version=\"1.0\" encoding=\"utf-8\"?><hsmbs><url>"++ Xykcx_url ++"</url><openTimer>true</openTimer><tranCode>form10071</tranCode></hsmbs>";
                       ["form50600"] ->
                                   Group2 = ?ewp_xpath("hsmbs/body/relacc/group", Doc),
                                   Cardindex = ?ewp_xpath("hsmbs/body/cardindex/value", Doc),
                                   Listno = lists:map(fun ({_,Group3}) ->
                                       Acc2 = proplists:get_value(acc,Group3),
                                       Aname = proplists:get_value(aname,Group3),
                                       CardList = "\{\"cardNo\":" ++ "\"" ++ Acc2 ++ "\",\"alias\":"++ "\"" ++ Aname ++"\"\},",
                                       CardList
                                   end,Group2),
                                   Listno_val = "\{\"data\":[" ++ Listno ++ "]\}",
                                   Listno_val2 = re:replace(Listno_val,",]\}","]\}",[{return,list},global]),
                                   Xykcx = cib_controller:new_xykcx(Listno_val2,"xyksz"),
                                   Xykcx_url = Xykcx ++ "&accountIndex=" ++ Cardindex,
                                   "<?xml version=\"1.0\" encoding=\"utf-8\"?><hsmbs><url>"++ Xykcx_url ++"</url><tranCode>form10071</tranCode></hsmbs>";
                       ["form60002"] -> %%银行卡管理--判断卡列表是否有调整
                           [Do_flag] = xml_eng:xpath_term_value("hsmbs/body/do_flag/value", Doc),
                           case Do_flag of
                               "1" ->
                                   Cardlist = xml_eng:xpath_term_value("hsmbs/body/relacc", Doc),
                                   {SessionId, Session} = session_service:read(),
                                   Session1 = lists:keystore(cardlist,1, Session,{cardlist,Cardlist}),
                                   session_service:update(SessionId, Session1);
                               _->
                                   go_on
                           end,
                           % update_jsession(?ewp_xpath("/hsmbs/jsessionid",Doc), AdapterName),
                           Res;
                       _ ->
                           % update_jsession(?ewp_xpath("/hsmbs/jsessionid",Doc), AdapterName),
                           Res
                           
                   end
           end,
       ResBody1 = re:replace(ResBody, "^[ \n]+", "",[global,{return, list}]),
       ResBody2 = re:replace(ResBody1, "[ \n]+$", "", [global,{return, list}]),
       ResBody2.
     
   %%模拟报文生成接口
   fangzy(_TranCode, _Channel) ->
       case file:consult("/var/www/apps/fangzy/script/conf/fang.conf") of
           {ok,Param} ->
               CibConf = proplists:get_value(cib_conf, Param),
               Dir = proplists:get_value(dir, CibConf),%XML报文路径
               Mode = proplists:get_value(mode, CibConf),%模块名
               testTrandCode= proplists:get_value(testTrandCode, CibConf),%交易代码
               FullName = Dir ++ "/" .. Mode ++ "/" ++ testTrandCode,

               ?ewp_log("CibConf=[~p]~n Dir=[~p]~n FullName=[~p]~n", [ CibConf, Dir, FullName]),
               case FullName of
                   "" ->
                       throw("File is NULL");
                   undefined ->
                       throw("File is NULL");
                   _ ->
                       go_on
               end,

               XmlContent =
                   case file:read_file(FullName) of
                       {ok, TmpXmlContent} ->
                           erlang:binary_to_list(TmpXmlContent);
                       _   ->
                           throw("Read File Failue")
                   end,

               list_to_binary(ebank_utils:xml_to_json(xmlFormate(XmlContent)));
           {error, ErrMsg} ->
               throw( file:format_error(ErrMsg))
       end.
   ```

3. ##### 实现方法

    通过读配制文件中的路径、模块和对应的交易代码拼成一个完整的路径，然后去读对应文件中的内容，该文件中的内容为对应交易的报文，再将取到的报文调用xmlFormate方法进行格式化，这个方法是ewp项目中cib/src/lib/ebank_utils.erl文件中已经存在的方法，保证返回的结果和请求接口返回的一致，然后将格式化后的内容转为json数据返回到对应的页面。

   报文样例:

   ```
   <?xml version="1.0"  encoding="utf-8"?>
   <hsmbs>
   	<jsessionid>ohGWy2HtlTYdFY2_ldVccR0OfOX3lARDPzaGwnpfw_XGC3hl46BW!-593322696!1527243301357</jsessionid>
     	<model>
   		<title>app动户通知</title>
   		<safeExitUrl>exit.cli</safeExitUrl>
   		<backUrl>60003.cli</backUrl>
   		<type>app</type>
   	</model>
   	<hidden>
   		<loginType>0</loginType>
   		<areaNo>11</areaNo>
   		<currentAcc>11901331291</currentAcc>
   	</hidden>
   	<body>
   	<cardList><group><acc><name>借记卡</name><value>622908 ****** *1291*</value></acc><acctype><name>卡类型</name><value>1</value></acctype><aname><name>账户别名</name><value>朱蓓</value></aname><khxm><name>客户姓名</name><value>马官员马官员马官员马官员马官员马官员马官马官员马员</value></khxm><kyye><name>人民币活期余额</name><value>22.24</value></kyye><url>50000.cli?flowsn=0&amp;listcardno=0</url><selected>true</selected></group></cardList>
   			<haveapp><name>有无app动户通知(1 有)</name><value></value></haveapp>
   	<list2>
   								<group><name>签约手机号</name><value>185****2423</value></group>		
   					</list2>
   	
   	<havedx><name>有无dx动户通知(1 有)</name><value></value></havedx>
   	<list1>
   					</list1>
   	<havewx><name>有无wx动户通知(1 有)</name><value></value></havewx>
   	<page_50000_1001><name>page_50000_1001</name><value>您尚未开通动户通知服务</value></page_50000_1001>
   	<page_50000_1002><name>page_50000_1002</name><value>无效的URL</value></page_50000_1002>
   	<ktdhtzurl><name>开通动户通知</name><value>50000.cli?flowsn=508</value></ktdhtzurl>
   	<xgdhtzurl><name>修改动户通知</name><value>50000.cli?flowsn=540</value></xgdhtzurl>
   		</body>
   	<tranCode>form50000_1</tranCode>
   </hsmbs>
   ```

4. ##### 请求方法

   ```
   ert.channel:first_page("fangzy", "fangzy", {id="fangzy",tranCode= "fangzy", testTranCode="form50000_1"}, {nopush=true,alertErr=false});
   ```

   testTranCode为对应的交易代码。

