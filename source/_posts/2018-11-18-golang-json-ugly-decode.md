---
layout: post
title: "GO语言解析那些乱七八糟JSON字符串的方法"
description: ""
category: 技术
tags: [go, json]
---

一些脚本语言对Json格式的要求比较松散，导致他们所提供了接口输出的数据也是一样的，可能存在以下这样子的情况：

* 相同的字段，类型是不固定的
* 将数据包裹在一个数组时，数组元素的类型也是不固定的

而Go语言本身是强类型的，这给解析这样子的数据造成了麻烦，你当然可以使用`interface{}`挨个去解析，但随着数据量大了之后，嵌套格式越深，这是一个无底洞。。。

这里将介绍一些go语言json解析的奇技淫巧，让你免遭痛楚。

<!-- more -->

# 情景一：相同字段，包含不同的类型

看下方这个Json字符串

```json
[
    {
        "plan_a": true,
        "plan_b": false
    },
    {
        "plan_a": 1542511080,
        "plan_b": 0
    }
]
```

这是一个特性的数组，表示用户是否激活PlanA或PlanB计划，这个plan_a字段可能是bool类型，或是一个时间戳。

最终，我在程序上，只需要知道用户是否激活计划A，或激活计划B，我并不关注时间戳的事情，所以相应的代码是这样子定义的：

```go
type Feature struct {
	PlanA bool `json:"plan_a"`
	PlanB bool `json:"plan_b"`
}
```

假定你现在就使用这样子的方法去解析Json：

```go
const featuresText = `
[
    {
        "plan_a": true,
        "plan_b": false
    },
    {
        "plan_a": 1542511080,
        "plan_b": 0
    }
]`

func main() {
	var features []Feature
	e := json.Unmarshal([]byte(featuresText), &features)
	log.Printf("features: %#v,\nerror: %v", features, e)
}
```

那么输出的结果是：

```txt
features: []main.Feature{main.Feature{PlanA:true, PlanB:false}, main.Feature{PlanA:false, PlanB:false}},
error: json: cannot unmarshal number into Go struct field Feature.plan_a of type bool
```

正确的解决方法是实现自定义Json解析的接口：

```go
type Feature struct {
	PlanA bool `json:"plan_a"`
	PlanB bool `json:"plan_b"`
}

// 定义一个丑陋的Feature，再使用switch type来解决类型的问题
type uglyFeature struct {
	UglyPlanA interface{} `json:"plan_a"` // 时间戳或布尔类型
	UglyPlanB interface{} `json:"plan_b"` // 时间戳或布尔类型
}

// 自定义Feature Json解码
func (f *Feature) UnmarshalJSON(b []byte) (e error) {
	var ugly uglyFeature
	e = json.Unmarshal(b, &ugly)
	if e != nil {
		return
	}
	// 是否已开通奖励计划A
	switch v := ugly.UglyPlanA.(type) {
	case bool:
		f.PlanA = v
	case float64:
		f.PlanA = v > 0
	}
	// 是否已开通奖励计划B
	switch v := ugly.UglyPlanB.(type) {
	case bool:
		f.PlanA = v
	case float64:
		f.PlanB = v > 0
	}
	return nil
}
```

相同的main函数，输出的结果是：

```txt
features: []main.Feature{main.Feature{PlanA:false, PlanB:false}, main.Feature{PlanA:true, PlanB:false}},
error: <nil>
```

# 情景二：数组的类型是不固定的

假定现在的数据更复杂了：

```json
{
    "result": [
        1,
        [
            {
                "device_sn": "SN0001",
                "feature": {
                    "plan_a": true,
                    "plan_b": false
                }
            },
            {
                "device_sn": "SN0002",
                "feature": {
                    "plan_a": 1542511080,
                    "plan_b": 0
                }
            }
        ]
    ]
}
```

而你的代码可能是这样子写的：

```go
func main() {
	var result struct {
		Result []interface{} `json:"result"`
	}
	e := json.Unmarshal([]byte(resultText), &result)
	log.Printf("result: %#v,\nerror: %v", result, e)
}
```

输出的结果是：

```
result: struct { Result []interface {} "json:\"result\"" }{Result:[]interface {}{1, []interface {}{map[string]interface {}{"device_sn":"SN0001", "feature":map[string]interface {}{"plan_a":true, "plan_b":false}}, map[string]interface {}{"device_sn":"SN0002", "feature":map[string]interface {}{"plan_a":1.54251108e+09, "plan_b":0}}}}},
error: <nil>
```

那些特性的数据，不是我们想要的类型`Feture`，而是`map[string]interface{}`，那么假定我想把它强制转为`Feture`，试试看：

```go
slice := result.Result[1].([]interface{})
	feature1 := slice[0].(map[string]interface{})["feature"]
	log.Printf("feature1: %#v", feature1) // feature1: map[string]interface {}{"plan_a":true, "plan_b":false}
	feature1cast := feature1.(Feature)
	log.Printf("feature1cast: %#v", feature1cast)
```

输出的结果是

```txt
feature1: map[string]interface {}{"plan_a":true, "plan_b":false}
panic: interface conversion: interface {} is map[string]interface {}, not main.Feature
```

呃，根本没办法转换，并且还涉及好多危险的类型转换，任意一个数据出错，都会导致程序panic，程序的健壮性非常差！

正确的做法和上边的差不多，不过更精妙一些：

```go
type Feature struct {
	PlanA bool `json:"plan_a"`
	PlanB bool `json:"plan_b"`
}

// 定义一个丑陋的Feature，再使用switch type来解决类型的问题
type uglyFeature struct {
	UglyPlanA interface{} `json:"plan_a"` // 时间戳或布尔类型
	UglyPlanB interface{} `json:"plan_b"` // 时间戳或布尔类型
}

// 自定义Feature Json解码
func (f *Feature) UnmarshalJSON(b []byte) (e error) {
	var ugly uglyFeature
	e = json.Unmarshal(b, &ugly)
	if e != nil {
		return
	}
	// 是否已开通奖励计划A
	switch v := ugly.UglyPlanA.(type) {
	case bool:
		f.PlanA = v
	case float64:
		f.PlanA = v > 0
	}
	// 是否已开通奖励计划B
	switch v := ugly.UglyPlanB.(type) {
	case bool:
		f.PlanA = v
	case float64:
		f.PlanB = v > 0
	}
	return nil
}

type Device struct {
	DeviceSN string  `json:"device_sn"`
	Feature  Feature `json:"feature"`
}

type Result struct {
	Code    int      `json:"code"`
	Devices []Device `json:"devices"`
}

// 自定义Result Json解码
func (p *Result) UnmarshalJSON(b []byte) (e error) {
	var code int
	var devices []Device
	var elements = []interface{}{&code, &devices} // must pass pointers
	e = json.Unmarshal(b, &elements)
	if e != nil {
		return
	}
	p.Code = code
	p.Devices = devices
	return
}
```

相应的main函数：

```go
func main() {
	var result struct {
		Result Result `json:"result"`
	}
	e := json.Unmarshal([]byte(resultText), &result)
	log.Printf("result: %#v,\nerror: %v", result, e)
}
```

相应的输出：

```txt
result: struct { Result main.Result "json:\"result\"" }{Result:main.Result{Code:1, Devices:[]main.Device{main.Device{DeviceSN:"SN0001", Feature:main.Feature{PlanA:false, PlanB:false}}, main.Device{DeviceSN:"SN0002", Feature:main.Feature{PlanA:true, PlanB:false}}}}},
error: <nil>
```

嗯，一切都符合预期，而整个解释的过程，核心关键是在于`var elements = []interface{}{&code, &devices}`，可以自己领悟一下。