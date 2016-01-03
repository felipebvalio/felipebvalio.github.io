---
layout:     post
title:      Testes Assíncronos
date:       2015-10-29 16:55
summary:    Como escrever testes unitários assíncronos
categories: jekyll
thumbnail:  cogs
tags:
 - testes unitários
 - swift
 - objective-c
---

Testes unitários precisam ser executados rapidamente, mas eventualmente é necessário realizar um teste assíncrono


Imagine o seguinte teste:

{% highlight swift %}
func testAsynchronous() {
    var finished = false
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, Int64(1 * Double(NSEC_PER_SEC))), dispatch_get_main_queue(), {
        finished = true
    })
    
    XCTAssertTrue(finished)
}
{% endhighlight %}

Ele falhará porque _XCTAssertTrue_ será executado antes da variável _finished_ ser atribuída com _true_. Podemos resolver este problema com a seguinte função:

{% highlight swift %}
func XCTWait(inout finished: Bool, timeout: NSTimeInterval) {
    let timeoutDate = NSDate(timeIntervalSinceNow: timeout)
    while !finished && timeoutDate.earlierDate(NSDate()) != timeoutDate {
        NSRunLoop.currentRunLoop().runMode(NSDefaultRunLoopMode, beforeDate: timeoutDate)
    }
    XCTAssertTrue(finished, "Test should complete in time.")
}
{% endhighlight %}

Que será utilizada da seguinte forma:

{% highlight swift %}
func testAsynchronous() {
    var finished = false
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, Int64(1 * Double(NSEC_PER_SEC))), dispatch_get_main_queue(), {
        finished = true
    })
    
    XCTWait(&finished, timeout: 3)
}
{% endhighlight %}

Este teste concluirá com sucesso após 1 segundo, que é quando _finished_ será atribuído com _true_. Caso essa atribuição não ocorra, o teste falhará depois de 3 segundos.

Mas e em Objective-C, como fica? Vamos construir uma função semelhante, porém com algumas peculiaridades:

{% highlight swift %}
#define XCTWait(finished, timeout) \
waitFor(&finished, timeout); \
XCTAssertTrue(finished, @"Test should complete in time.");

void waitFor(BOOL *finished, NSTimeInterval timeout) {
    NSDate *timeoutDate = [NSDate dateWithTimeIntervalSinceNow:timeout];
    while (!*finished && [timeoutDate earlierDate:[NSDate date]] != timeoutDate) {
        [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:timeoutDate];
    }
}
{% endhighlight %}

Que, por fim, será utilizado no teste da seguinte maneira:

{% highlight swift %}
- (void)testAsynchronous {
    __block BOOL finished = NO;
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 1 * NSEC_PER_SEC), dispatch_get_main_queue(), ^{
        finished = YES;
    });
    
    XCTWait(finished, 3);
}
{% endhighlight %}
