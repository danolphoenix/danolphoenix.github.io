---
layout: post
title: ceilometer 常用命令行
tags: meter, ceilometer
categories: 工作记录，ceilometer 
---

# ceilometer 常用命令行

> * **[meter-list](#meter-list)**       List the user's meters .获取监控维度列表
> * **[query-samples](#query-samples)**       Query samples.筛选采样样本
> * **[resource-list](#resource-list)**       List the resources.获取监控资源列表
> * **[resource-show](#resource-show)**        Show the resource.获取监控资源详情
> * **[sample-create](#sample-create)**       Create a sample创建采样样本
> * **[sample-list](#sample-list)**      List the samples (return OldSample objects if  -m/--meter is set).获取采样样本列表
> * **[sample-show](#sample-show)**                 Show an sample.显示采样样本详情
> * **[statistics](#statistics)**                  List the statistics for a meter.获取监控维度统计数据

<span id="meter-list"> </span>
### 1.  ceilometer meter-list

List the user's meters. 获取监控维度列表
usage: 
**ceilometer meter-list [-q `<QUERY>` ]**

- Optional arguments:
`  -q <QUERY>, --query <QUERY>`  key[op]data_type::value; list. data_type is optional, but if supplied must be string,integer, float, or boolean.
  
  > -q 参数支持如下查询项：['pagination', 'project', 'resource', 'source','user']，
  > **`支持对resource字段的通配查询`**
  
  @(FIXME(hzluodan))[pagination在服务端其实还未实现]
 
- 使用示例:

  > $ ceilometer meter-list -q resource=.*netease.10-180-0-33.mem.used
    **发往API:**
curl -g -i -X 'GET' 'http://netbetaapi.beta.server.163.org:8777/v2/meters
 
- 返回示例
 > +------------------------------+-------+------+-------------------------------------------------------------------+----------------------------------+----------------------------------+
| Name                         | Type  | Unit | Resource ID                                                       | User ID                          | Project ID                       |
+------------------------------+-------+------+-------------------------------------------------------------------+----------------------------------+----------------------------------+
| netease.10-180-0-33.mem.used | gauge | Mb   | 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.mem.used | 2d0bc491612d4701ab24e3df862ca9ab | 6db0dde8b5704f60b5fb21f12da52b45 |
| netease.10-180-0-33.mem.used | gauge | kB   | 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.mem.used | 2d0bc491612d4701ab24e3df862ca9ab | 6db0dde8b5704f60b5fb21f12da52b45 |
+------------------------------+-------+------+-------------------------------------------------------------------+----------------------------------+----------------------------------+

<span id="query-samples"> </span>
### 2.  ceilometer query-samples[ 略难用，不建议推广 ]
Query samples.筛选采样样本，和sample-list差不多，表达式语义比较难用，跟sample-list -q后接查询条件相比相比，有个好处是可以按count_volume值来进行筛选，这个是sample-list -q所没有的功能。

usage: 
**ceilometer query-samples [-f `<FILTER>`] [-o `<ORDERBY>`] [-l `<LIMIT>`]**

- Optional arguments:
>  `-f <FILTER>, --filter <FILTER>`
                                {complex_op: [{simple_op: {field_name: value}}]} 
                                The complex_op is one of: ['and','or'], simple_op is one of: ['=', '!=', '<', '<=', '>', '>='].
 ` -o  <ORDERBY>, --orderby  <ORDERBY>`
                                [{field_name: direction}, {field_name:  direction}] The direction is one of: ['asc', 'desc'].
 ` -l  <LIMIT> , --limit  <LIMIT>`   Maximum number of samples to return.
  


- 使用示例:

 > 在开发机上
1. ceilometer  query-samples -f '{"and":[{"=":{"counter_name":"cpu"}},{">":{"counter_volume":100}} ]}' -l 10

 > 在联调环境上
2. ceilometer --debug query-samples -f '{"and":[{"=":{"counter_name":"netease.10-180-0-33.cpu.rate"}},{"<":{"counter_volume":100}}]}' -l 10
3. ceilometer --debug query-samples -f '{"and":[{"=":{"counter_name":"netease.10-180-0-33.cpu.rate"}},{"<":{"counter_volume":100}},{">":{"timestamp":"2015-12-28T08:08:37"}}, {"<": {"timestamp": "2015-12-28T08:24:37"}}]}' -l 10

 **发往API**
curl -g -i -X 'POST' 'http://netbetaapi.beta.server.163.org:8777/v2/query/samples' -H 'User-Agent: ceilometerclient.openstack.common.apiclient' -H 'Content-Type: application/json' -H 'X-Auth-Token: {SHA1}f3462a032257c34ccd996465f8c15d0fc8287156'
DEBUG (client) REQ BODY: {"filter": "{\"and\":[{\"=\":{\"counter_name\":\"netease.10-180-0-33.cpu.rate\"}},{\"<\":{\"counter_volume\":100}}]}", "limit": "10"}


 
- 返回示例
> +-------------------------------------------------------------------+------------------------------+-------+----------------+------+---------------------+
| Resource ID                                                       | Meter                        | Type  | Volume         | Unit | Timestamp           |
+-------------------------------------------------------------------+------------------------------+-------+----------------+------+---------------------+
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.408571666806 | %    | 2015-12-28T08:22:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.450300200133 | %    | 2015-12-28T08:20:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.441519493502 | %    | 2015-12-28T08:18:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.43452828612  | %    | 2015-12-28T08:16:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.441078561917 | %    | 2015-12-28T08:14:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.442256341789 | %    | 2015-12-28T08:12:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.440968466595 | %    | 2015-12-28T08:10:37 |
+-------------------------------------------------------------------+------------------------------+-------+----------------+------+---------------------+

<span id="resource-list"> </span>
### 3.  ceilometer resource-list
List the resources. 获取监控资源的列表,
返回所有当前被监控的资源，也是返回resource表中的内容，因为现在resource表一条记录对应一个uuid+维度，该命令返回的内容和meter-list一样。返回结果后文中有，这里不贴了。

usage: **ceilometer resource-list [-q `<QUERY>`]**
Optional arguments:


 >  `-q <QUERY>, --query <QUERY>`  key[op]data_type::value; list. data_type is
                               optional, but if supplied must be string,
                               integer, float, or boolean.
                               
> -q 参数支持 ['pagination', 'project', 'resource', 'search_offset', 'source', 'timestamp', 'user']


- 使用示例:
 > $ ceilometer resource-list
**发往API**:
curl -g -i -X 'GET' 'http://netbetaapi.beta.server.163.org:8777/v2/resources' -H 'User-Agent: ceilometerclient.openstack.common.apiclient' -H 'X-Auth-Token: {SHA1}f7ec72591f7ae747fd85e2c5e0fa2585f35339b1'

<span id="resource-show"> </span>
### 4. ceilometer resource-show      
Show the resource. 显示被监控资源的资源详情     
    
usage:
 **ceilometer resource-show `<RESOURCE_ID>`**

Positional arguments:
  `<RESOURCE_ID>`:   ID of the resource to show.
- 使用示例:
> $ ceilometer --debug resource-show f7d752c0-05f2-43f3-aba2-9fdcc5c8834c.netease.10-180-0-38.nic.eth1.bad9fe29-664e-4097-a906-b2704a9b0ed4.10-180-169-68.trans.rate
**发往API**
curl -g -i -X 'GET' 'http://netbetaapi.beta.server.163.org:8777/v2/resources/f7d752c0-05f2-43f3-aba2-9fdcc5c8834c.netease.10-180-0-38.nic.eth1.bad9fe29-664e-4097-a906-b2704a9b0ed4.10-180-169-68.trans.rate' -H 'User-Agent: ceilometerclient.openstack.common.apiclient' -H 'X-Auth-Token: {SHA1}0139011442a0bab0fe3bf1097cbddb3353ea1c5d'

- 返回示例
>+-------------+--------------------------------------------------------------------------+
| Property      | Value                                                                                                        |
+-------------+--------------------------------------------------------------------------+
| metadata    | {}                                                                                                              |
| project_id  | 6db0dde8b5704f60b5fb21f12da52b45                                             |
| resource_id | f7d752c0-05f2-43f3-aba2-9fdcc5c8834c.netease.10-180-0-38.nic.eth1.bad9fe |
|                     | 29-664e-4097-a906-b2704a9b0ed4.10-180-169-68.trans.rate                  |
| source      | openstack                                                                                                  |
| user_id     | 2d0bc491612d4701ab24e3df862ca9ab                                               |
+-------------+------------------------------------------------------------------------- -+

<span id="sample-create"> </span>
### 5.ceilometer sample-create   

Create a sample.
####`创建采样样本，这个功能应该不使用，不然谁都能推样本了，大乱。`
usage:  **ceilometer sample-create **` [ -- project-id  <SAMPLE_PROJECT_ID> ] 
                                [- -user-id  <SAMPLE_USER_ID> ]  -r <RESOURCE_ID>  
                                - m <METER_NAME> --meter-type  <METER_TYPE>   
                                 --meter-unit  <METER_UNIT>  --sample-volume <SAMPLE_VOLUME>  
                               [--resource-metadata <RESOURCE_METADATA> ]  
                               [--timestamp  <TIMESTAMP> ]` 



Optional arguments:
   > `--project-id <SAMPLE_PROJECT_ID>`
                                Tenant to associate with sample (only settable
                                by admin users).
 ` --user-id <SAMPLE_USER_ID>`    
 User to associate with sample (only settable
                                by admin users).
 `-r <RESOURCE_ID>, --resource-id <RESOURCE_ID>`
                                ID of the resource. Required.
  `-m <METER_NAME>, --meter-name <METER_NAME>`
                                The meter name. Required.
  `--meter-type <METER_TYPE>`
       The meter type. Required.
  `--meter-unit <METER_UNIT>`     
  The meter unit. Required.
  `--sample-volume <SAMPLE_VOLUME>`
                                The sample volume. Required.
  `--resource-metadata <RESOURCE_METADATA>`
                                Resource metadata. Provided value should be a
                                set of key-value pairs e.g. {"key":"value"}.
  `--timestamp <TIMESTAMP>`      
   The sample timestamp.
   
-使用示例:
>  在开发机上：
ceilometer --debug sample-create -r a014508c-e0fe-42c9-a6c5-dcd4cd0ae1f8 -m cpu --meter-type delta --meter-unit bps --sample-volume 999
**发往API**:
 REQ: curl -g -i -X 'POST' 'http://10.166.224.20:8777/v2/meters/cpu' -H 'User-Agent: ceilometerclient.openstack.common.apiclient' -H 'Content-Type: application/json' -H 'X-Auth-Token: {SHA1}6d583ff31d7f73d302b413a4f61ed6de220c9348'
DEBUG (client) REQ BODY: [{"counter_type": "delta", "counter_name": "cpu", "resource_id": "a014508c-e0fe-42c9-a6c5-dcd4cd0ae1f8", "counter_unit": "bps", "counter_volume": "999"}]

- 响应示例：
> 返回创建的采样样本
+-------------------+--------------------------------------------+
| Property                | Value                                      |
+-------------------+--------------------------------------------+
| message_id          | 1e535362-ad44-11e5-956c-fa163e35b12a       |
| name                    | cpu                                        |
| project_id             | 9651062b50644c83adf46643791dab9c           |
| resource_id           | a014508c-e0fe-42c9-a6c5-dcd4cd0ae1f8       |
| resource_metadata | {}                                         |
| source                  | 9651062b50644c83adf46643791dab9c:openstack |
| timestamp           | 2015-12-28T09:19:38.274449                   |
| type                     | delta                                      |
| unit                     | bps                                        |
| user_id                 | d8cf012507f64826a5ce5ecf9c2a2e9f           |
| volume                | 999.0                                      |
+-------------------+--------------------------------------------+


<span id="sample-list"> </span>
### 6. ceilometer sample-list    
List the samples (return OldSample objects if -m/--meter is set).获取采样值列表


usage: 
**ceilometer sample-list `[-q <QUERY>] [-m <NAME>] [-l <NUMBER>]`**


Optional arguments:
` -q <QUERY>, --query <QUERY>`   key[op]data_type::value; list. data_type is
                                optional, but if supplied must be string,
                                integer, float, or boolean.
 ` -m <NAME>, --meter <NAME> `    Name of meter to show samples for.
`  -l <NUMBER>, --limit <NUMBER>`    Maximum number of samples to return.

-q参数支持:['message_id', 'meter', 'project', 'resource', 'search_offset', 'source', 'timestamp', 'user']

获取监控采样样本，注意和query-sample还是有差别，目前发现的就是对counter_volume值的筛选，sample-list不支持，而query-sample可以通过组合条件之间的关系支持。

注意-m选项后面接的是不带uuid的meter_name。

- 使用示例:
> hzluodan@10-180-0-24:~$ ceilometer sample-list -m netease.10-180-0-33.cpu.rate -q resource=08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate -l 10
>**发往api**
>curl -g -i -X 'GET' 'http://10.166.224.20:8777/v2/samples?limit=10

- 响应示例
>+-------------------------------------------------------------------+------------------------------+-------+----------------+------+---------------------+
| Resource ID                                                       | Name                         | Type  | Volume         | Unit | Timestamp           |
+-------------------------------------------------------------------+------------------------------+-------+----------------+------+---------------------+
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.408435442194 | %    | 2015-12-29T02:26:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.424646128226 | %    | 2015-12-29T02:24:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.400667779633 | %    | 2015-12-29T02:22:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.425106276569 | %    | 2015-12-29T02:20:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.44140917798  | %    | 2015-12-29T02:18:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.425141713905 | %    | 2015-12-29T02:16:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.418060200669 | %    | 2015-12-29T02:14:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.433477825942 | %    | 2015-12-29T02:12:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.416840350146 | %    | 2015-12-29T02:10:37 |
| 08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.cpu.rate | netease.10-180-0-33.cpu.rate | gauge | 0.431893687708 | %    | 2015-12-29T02:08:37 |
+-------------------------------------------------------------------+------------------------------+-------+----------------+------+---------------------+

<span id="sample-show"> </span>
###7 ceilometer sample-show       
Show an sample.
显示采样值详情。估计用不上吧，谁会去找message_id这么个难搞的数值
          
usage: 
**ceilometer sample-show `<SAMPLE_ID>`**



Positional arguments:
`  <SAMPLE_ID> ` ID (aka message ID) of the sample to show.



<span id="statistics"> </span>
###8 ceilometer statistics  
List the statistics for a meter. 获取监控维度的统计数据
usage:
 **ceilometer statistics` [-q <QUERY>] -m <NAME> [-p <PERIOD>] [-g <FIELD>] [-a <FUNC>[<-<PARAM>]]`**

Optional arguments:
  `-q <QUERY>, --query <QUERY> `  key[op]data_type::value; list. data_type is
                                optional, but if supplied must be string,
                                integer, float, or boolean.
`  -m <NAME>, --meter <NAME>  `   Name of meter to list statistics for.
                                Required.
`  -p <PERIOD>, --period <PERIOD>`
                                Period in seconds over which to group samples.
 ` -g <FIELD>, --groupby <FIELD>`
                                Field for group by.对聚合结果的分组依据
 ` -a <FUNC>[<-<PARAM>], --aggregate <FUNC>[<-<PARAM>]`
                                Function for data aggregation. Available
                                aggregates are: count, cardinality, min, max,
                                sum, stddev, avg. Defaults to [].对聚合样本所使用的公式。

`-q 参数支持` : ['message_id', 'meter', 'project', 'resource', 'search_offset', 'source', 'timestamp', 'user']
- 使用示例：
> $ ceilometer --debug statistics 
> `-m netease.10-180-0-33.nic.eth0.1967defb-715b-4585-9b90-cb85437e2cdf.10-180-64-38.recv.rate `
> `-q "resource=08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.nic.eth0.1967defb-715b-4585-9b90-cb85437e2cdf.10-180-64-38.recv.rate;timestamp<2015-12-29T02:34:37;timestamp>2015-12-28T01:34:37"`

**发往api**
 curl -g -i -X 'GET' 'http://netbetaapi.beta.server.163.org:8777/v2/meters/netease.10-180-0-33.nic.eth0.1967defb-715b-4585-9b90-cb85437e2cdf.10-180-64-38.recv.rate/statistics?q.field=resource&q.field=timestamp&q.field=timestamp&q.op=eq&q.op=lt&q.op=gt&q.type=&q.type=&q.type=&q.value=08407aa0-71ef-43c1-b1b0-c07255d166ca.netease.10-180-0-33.nic.eth0.1967defb-715b-4585-9b90-cb85437e2cdf.10-180-64-38.recv.rate&q.value=2015-12-29T02%3A34%3A37&q.value=2015-12-28T01%3A34%3A37' -H 'User-Agent: ceilometerclient.openstack.common.apiclient' -H 'X-Auth-Token: {SHA1}938ed4b11ba58b7a2d0fbce16d0b372272bc74ac'

- 响应示例：
>+--------+---------------------+---------------------+---------------+---------------+---------------+---------------+-------+----------+---------------------+---------------------+
| Period | Period Start        | Period End          | Max           | Min           | Avg           | Sum           | Count | Duration | Duration Start      | Duration End        |
+--------+---------------------+---------------------+---------------+---------------+---------------+---------------+-------+----------+---------------------+---------------------+
| 0      | 2015-12-28T01:36:36 | 2015-12-29T02:34:36 | 109.345685493 | 35.0965046379 | 87.2659142026 | 65449.4356519 | 750   | 89880.0  | 2015-12-28T01:36:36 | 2015-12-29T02:34:36 |
+--------+---------------------+---------------------+---------------+---------------+---------------+---------------+-------+----------+---------------------+---------------------+



###命令行参数指定tips:
1. 在-q中指定多个过滤条件的方法：用分号连接，各个过滤条件之间是与的关系。
若要用并关系或者按counter_volume取值查询，目前估计只能用query-sample 这个CLI
`ceilometer …… -q "resource=xxx ;  timestamp<yyy"`

2. 目前只有在访问meter表的代码里用到get_meter方法中才支持通配, 即
>  $ ceilometer meter-list -q resource=通配表达式

  其余的-q参数中虽然指定resource的值，但是最后没有落到get_meter方法来处理，还是无法通配。
3. 带 `-q <QUERY>`的后接参数不同命令支持的-q范围不一样，大抵因为访问的表不同。
4. -q 后接参数中的`resource=xxx`，其中的xxx是`uuid+维度名的形式`
5. -m 后接的`meter_name`，只是维度名，不含uuid
6. statistics指定多个groupby参数作为分组依据的方法：
`ceilometer statistics  …… -g "project_id"  -g "user_id"`
7. statistics对于cardinality公式的参数给法示例：
` ceilometer statistics -m cpu -a 'cardinality<-project_id'`

参考文档：

[ceilometer query-sample 的参数使用方法 https://wiki.openstack.org/wiki/Ceilometer/ComplexFilterExpressionsInAPIQueries](https://wiki.openstack.org/wiki/Ceilometer/ComplexFilterExpressionsInAPIQueries)


[ceilometer api v2文档 
http://docs.openstack.org/developer/ceilometer/webapi/v2.html ](http://docs.openstack.org/developer/ceilometer/webapi/v2.html)

 
