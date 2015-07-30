# Data

下面列出了每个`dropwizard metric`在`JSON`中的表现形式. 对于每个`metric`详细信息请参考[Dropwizard metrics documentation](http://www.gnim.wang/2015/07/07/dropwizard/)

### Gauge
```json
{
  "value" : value // any json value
}
```

### Counter
```json 
{
  "count" : 1 // number
}
```

### Histogram
```json 
{
  "count"  : 1 // long
  "min"    : 1 // long
  "max"    : 1 // long
  "mean"   : 1.0 // double
  "stddev" : 1.0 // double
  "median" : 1.0 // double
  "75%"    : 1.0 // double
  "95%"    : 1.0 // double
  "98%"    : 1.0 // double
  "99%"    : 1.0 // double
  "99.9%"  : 1.0 // double
}
```

### Meter
```json 
{
  "count"             : 1 // long
  "meanRate"          : 1.0 // double
  "oneMinuteRate"     : 1.0 // double
  "fiveMinuteRate"    : 1.0 // double
  "fifteenMinuteRate" : 1.0 // double
  "rate"              : "events/second" // string representing rate
}
```

### ThroughputMeter
扩展自`Meter`,提供即时吞吐量。
```json 
{
  "count"             : 40 // long
  "meanRate"          : 2.0 // double
  "oneSecondRate"     : 3 // long - number of occurence for the last second
  "oneMinuteRate"     : 1.0 // double
  "fiveMinuteRate"    : 1.0 // double
  "fifteenMinuteRate" : 1.0 // double
  "rate"              : "events/second" // string representing rate
}
```

### Timer
`Histogram`和`Meter`的组合：

```json 
{
  // histogram data
  "count"  : 1 // long
  "min"    : 1 // long
  "max"    : 1 // long
  "mean"   : 1.0 // double
  "stddev" : 1.0 // double
  "median" : 1.0 // double
  "75%"    : 1.0 // double
  "95%"    : 1.0 // double
  "98%"    : 1.0 // double
  "99%"    : 1.0 // double
  "99.9%"  : 1.0 // double

  // meter data
  "meanRate"          : 1.0 // double
  "oneMinuteRate"     : 1.0 // double
  "fiveMinuteRate"    : 1.0 // double
  "fifteenMinuteRate" : 1.0 // double
  "rate"              : "events/second" // string representing rate
}
```

### Throughput Timer
拓展自`Timer`,提供即时吞吐量的`metric`

```json 
{
  // histogram data
  "count"      : 1 // long
  "min"        : 1 // long
  "max"        : 1 // long
  "mean"       : 1.0 // double
  "stddev"     : 1.0 // double
  "median"     : 1.0 // double
  "75%"        : 1.0 // double
  "95%"        : 1.0 // double
  "98%"        : 1.0 // double
  "99%"        : 1.0 // double
  "99.9%"      : 1.0 // double

  // meter data
  "meanRate"          : 1.0 // double
  "oneSecondRate"     : 3 // long - number of occurence for the last second
  "oneMinuteRate"     : 1.0 // double
  "fiveMinuteRate"    : 1.0 // double
  "fifteenMinuteRate" : 1.0 // double
  "rate"              : "events/second" // string representing rate
}
```