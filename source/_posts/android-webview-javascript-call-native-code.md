---
layout: post
date: 2018-02-08 00:00:00
title: "Web JavaScript调用Android原生应用代码"
description: ""
category: 技术
tags: 
  - Android
  - md5sum
---

使WebView可以调用原生应用的代码，通过注册Event事件的形式

简单的调用方法是：

```
<button id="btn_test_event">测试事件</button>
<script type="text/javascript">
document.getElementById(btn_test_event).onclick = function () {
    var command = {
        'action': 'test'
    };
    Eagle.event(JSON.stringify(command));
}
</script>
```

这其中的Eagle就是客户端指定的`addJavascriptInterface(EagleWebInterface.create(this), "Eagle")`
而event是指event()这个方法:

```
@JavascriptInterface
public String event(String params) {
    final String action = JSON.parseObject(params).getString("action");
    final Event event = EventManager.getInstance().createEvent(action);
    Logger.d("event: " + params);
    if (event != null) {
        event.setAction(action);
        event.setDelegate(DELEGATE);
        event.setContext(DELEGATE.getContext());
        event.setUrl(DELEGATE.getUrl());
        return event.execute(params);
    }
    return null;
}
```

