<!--
{
"name":"20150403",
"author": "ckeyer",
"head": "http://moefq.com/images/2015/11/23/2341564017cc8b9a8e6a19963f82125b.png",
"date": "2015-04-03",
"title": "HashMap和TreeMap",
"tags": ["数据结构", "Hash"],
"category": ["学习笔记"],
"status": "publish",
"summary": "简单介绍一下HashMap和TreeMap的实现和区别。"
}
-->


首先什么是Map。说白了，就是键值对。在数组中我们是通过数组下标来对其内容索引的，而在Map中我们通过对象来对对象进行索引，用来索引的对象叫做key，其对应的对象叫做value。

HashMap通过hashcode对其内容进行快速查找，而TreeMap中所有的元素都保持着某种固定的顺序，如果你需要得到一个有序的结果你就应该使用TreeMap（HashMap中元素的排列顺序是不固定的）。


