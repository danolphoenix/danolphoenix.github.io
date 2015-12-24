---
layout: post
title: 一个meter在ceilometer中的漂流
tags: meter, ceilometer
categories: 工作记录，ceilometer 
---

Openstack的ceilometer模块主要用于数据的采集（包括监控数据，计费数据等）来提供给监控、计费、前端展示所使用。

在Openstack的ceilometer模块中，对某个资源（resource）的计量项，称之为一个meter。
在某个时刻对某个资源（resource）的某个计量项（meter）的采样数据，称之为一个sample。


![enter image description here](http://images.cnitblog.com/blog2015/697113/201504/011557010923003.jpg)

如上图所示，一个采样数据sample由生到灭的过程包括：对预定的测量项Meter进行获取(get_sample)、经过Transfomer进行转换(_transform_sample)，然后由Publiser进行发布(publish_samples)。

现从源码层面来分析下该流程，以对云主机的cpu信息进行测量采样为例：

###**前期准备工作：**

需要对获取、转换、和发布的三个步骤进行预先配置，这也是ceilometer灵活的一面。通过不同的配置项可以自由地指定想要的处理动作。
####获取（get_sample)部分：
   对云主机的信息采样主要由ceilometer-agent-compute来执行，因此在ceilometer/setup.cfg中的ceilometer.poll.compute段中写明了 
```python
     cpu = ceilometer.compute.netease_pollsters.cpu:CPUPollster
```
   意为使用CPUPollster来负责具体的获取(get_sample) 动作。
   
####转换(_transform_sample)、发布(publish_samples)部分

   Ceilometer是根据配置文件/etc/ceilometer/pipeline.yaml来配置计量项(meters)所使用的transformer和publisher的。以一个CPU meter为例：
   
```python
sources:  # A source is a producer of samples。
     - name: cpu_source #测量项目的名字
      interval: 600     #为该测量项设定的测量时间间隔，此处interval为600秒
      meters:
          - "cpu" # 由它来执行测量动作，具体的执行者，在上面的setup.cfg中已经有写明。
      sinks: # 指明对该sample数据所要使用的后续处理
          - cpu_sink # 在下面的sink段中有具体声明

sinks:
    - name: cpu_sink # 这里定义了上面的那个sinks，sink可以想像为对采样数据的后续处理步骤集合
      transformers:# 指定了该sink所使用的transformer，这里不填表示不用转换，直接发布，但对于平均值，波动值等需要依赖之前数据的采样结果，就需要指定transformer
      publishers:# 指定了所使用publisher，此处使用notifier意为将之转为message发布到AMQP中，
          - notifier://
```

然后开始：

1.数据采集：
   当ceilometer-agent-compute启动的时候，会给它设置好上面那个定时任务，即每隔600秒执行一次采样cpu数据。
   每当时间到时，会自动触发agent基类agent/base.py中的poll_and_publish（）方法。

   在该方法中，先利用novaclient向nova收集了一些虚拟机的基本数据，包括uuid等。然后将这些基本数据做成一个list传给ceilometer.compute.netease_pollsters.cpu:CPUPollster中的get_Samples()方法。

```python
class CPUUtilPollster(pollsters.BaseComputePollster):
    def get_samples(self, manager, cache, resources):
        self._inspection_duration = self._record_poll_time()
        for instance in resources:
            try:
                cpu_info = self.inspector.inspect_cpu_util(
                    instance, self._inspection_duration) # 获取cpu信息
                yield util.make_sample_from_instance(
                    instance,
                    name='cpu_util',
                    type=sample.TYPE_GAUGE,
                    unit='%',
                    volume=cpu_info.util,
                ) # 组装成Sample对象
```
在该方法中，会进一步向libvirt获取虚拟机的更具体细节的数据，比如我们想获取的cpu信息。最后将每台虚拟机的cpu的信息组装成一个Sample对象，再将包含所有虚拟机的cpu信息的Sample对象列表返回。
我有两台虚拟机，get_samples(）方法返回的Sample对象列表内容如下： 
```powershell
[<name: cpu, volume: 5364280000000, resource_id: a014508c-e0fe-42c9-a6c5-dcd4cd0ae1f8, timestamp: 2015-12-16T06:58:21Z>, 
<name: cpu, volume: 5381300000000, resource_id: 183278d3-fb75-4939-bd55-29efd351c8a1, timestamp: 2015-12-16T06:58:21Z>]
```

2.数据转换和发布
  
 然后这个Sample对象列表会被送给ceilometer.pipeline.Samplepipeline中的publish_data()方法。在这之中会先对是否支持该数据的下一步操作进行检查，如果可以，就把支持的数据送给所配置sink的publish_samples（）方法。在前文中已经说过，配置的sinks为cpu_sink。   
```python
class SampleSink(Sink):
    def _publish_samples(self, start, ctxt, samples):
        transformed_samples = []
        if not self.transformers: # 如果没有配置transformer的话，就直接跳过转换，将数据送到下面的publish过程。
            transformed_samples = samples
        else:
            for sample in samples:# 如果配置了transformer的话，就由_transform_sample进行转换，再将数据送到下面的publish方法。
                sample = self._transform_sample(start, ctxt, sample)
                if sample:
                    transformed_samples.append(sample)

        if transformed_samples:
            for p in self.publishers:
                try:
                    p.publish_samples(ctxt, transformed_samples) # 对Sample数据们进行publish。
                except Exception:
                   ...
```
在上面的p.publish_samples(ctxt, transformed_samples)方法中，由于在pipeline.yaml中配置的publisher是：
```python
- notifier://
```
因此采用的是RPC发布，也可以用其他publisher进行发布，或者配置多个publisher进行多渠道的发布，目前支持的有RPC，UDP和文件发布等。

**以RPC发布为例：具体执行发布的步骤详细如下：**

-  1.将Sample数据列表中包含的数据信息转换成一个用于发布或者存储的信息格式的metering message，就是将sample列表中每个sample中的具体数据提取成一个dict。最后形成的消息message中包含该dict列表。

- 2.将匹配的topic，message添加到本地队列local_queue中，topic在配置项中默认为'metering'；
- 3.将要发布的meters列表信息添加进self.local_queue，它一开始是一个空的list。
- 4.实现发布本地队列local_queue中的所有数据信息，在这之后会将message推送给AMQP，完成发布。

**代码细节为：**


发布sample
```python
    def publish_samples(self, context, samples):
        meters = [           
            utils.meter_message_from_counter(
                sample, cfg.CONF.publisher.telemetry_secret)
            for sample in samples
        ]# 上述步骤1，执行从Sample数据到消息格式数据的转换     
        topic = cfg.CONF.publisher_rpc.metering_topic #上述步骤2 
        self.local_queue.append((context, topic, meters)) #上述步骤3   
        if self.per_meter_topic: #下面这一小段不晓得做什么的
            ...
     self.flush() # 数据发布到消息队列中
```
之后包含了Sample数据的message就被推到AMQP里了，对它感兴趣的监听者（比如ceilometer-collector)可以对消息里的数据进行收集，再将数据存储到mongoDB中。

其他的用户就可以通过相应的API或者命令行，查询到这些采样数据。