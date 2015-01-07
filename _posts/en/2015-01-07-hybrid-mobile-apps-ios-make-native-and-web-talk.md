---
layout: post
lang: "en"
title: "Hybrid Mobile Apps in iOS: Making Native and Web Talk"
---

Deciding which techonology to use when building mobile app is often a binary choice between native and web. Both having pros and cons, which sometimes leads to combination: Hybrid apps. In a nuttshell, hybrid apps have a native layer and a web based layer (web apps always need at least a ``UIWebView`` to run on, hybrid apps bring more than just the ``UIWebView`` part).

Having responsibilites sepparated across these layers creates the need to share information between them. The web app needs to trigger something on the native code and vice versa.

#Native to Web
A typical use case for this is when app settings may change the web app behaviour. These need to be read when the application loads (or whenever the settings change) and then be sent to the web app.

##Javascript Events

Javascript events are the easiest way for a native app to send a message for the web app. ``UIWebView`` has a method called ``stringByEvaluatingJavaScriptFromString:(NSString *)script`` that can be used for this porpuse. An example to broadcast Angular event in Objective-C could look something like this:

```objective-c

NSString* eventToBroadCast = @"$rootScope.$broadcast('someEvent', someParameter);";
NSString* angularCode = [NSString stringWithFormat:@"var scope=angular.element(document.getElementById('main') || document.documentElement).scope(); if(scope){%@}", eventToBroadcast];

[self.webView stringByEvaluatingJavaScriptFromString:angularCode];
    
```

The event is then handled normally by the Angular application. In fact, the web app doesn't even know about the existance of a native code behind it. So how would the communication flow the other way around?

#Web to Native

Sending a message from native to web is fairly easy, the tricky part is the opposite direction: the web app is not aware that there is a native layer behind it. To all efects, the web app is simply running on a browser: be it Chrome on a PC or a ``UIWebView`` on an iOS app.

But the native app is aware of the web app existance and **it can intercept all the requests made by the web app**. That's how the communication works: the web app makes a request (containing the message) and the native app intercepts and responds to it.

##NSURLProtocol

``NSURLProtocol`` is the class that allows request to be intercepted before they actually reach the network. It also provides a way to give fake responses to the caller, and that's the mechanism used for communication between layers.

Here is a quick look on ``NSURLProtocol`` (only methods that are relevant to this context are shown):

```objective-c

@interface NSURLProtocol : NSObject
{
+ (void)registerClass;
+ (BOOL)canInitWithRequest:(NSURLRequest *)request;
+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request;
- (void)startLoading;
- (void)stopLoading;
}
```

**+ (void)registerClass** is used to register a new ``NSURLProtocol``. When a request is made by the ``UIWebView``, the native app iterates through all the registered interceptors asking if any is capable of handling the request. If not, the request goes directly to the network.

**+ (BOOL)canInitWithRequest:(NSURLRequest \*)request** must be implemented by subclass to indicate if it is capable of handling the request or not. The usual implementation is to search for a pattern in the requested url.

**+ (NSURLRequest \*)canonicalRequestForRequest:(NSURLRequest \*)request** this method should be implemented to make sure that, if two given URL are the same, they return the same ``NSURLRequest``. Normally it is ok to just return the request sent as a parameter.

**- (void)startLoading** this is where the magic happens. This method is called after the proper ``NSURLProtocol`` is found and created with the client request. From here, any native operation should be done or triggered and a ``NSHTTPURLResponse`` should be built with the results. This response is then passed to the client by calling ``- (void)URLProtocol:(NSURLProtocol *)protocol didReceiveResponse:(NSURLResponse *)response cacheStoragePolicy:(NSURLCacheStoragePolicy)policy;`` and ``- (void)URLProtocol:(NSURLProtocol *)protocol didLoadData:(NSData *)data;``. It's simpler than it looks, here is an example:

```objective-c
- (void)startLoading {
    NSData* body = [self getResponseBody];
    
    NSHTTPURLResponse *response = [[NSHTTPURLResponse alloc] initWithURL:self.request.URL
                                               statusCode:200
                                              HTTPVersion:@"1.1"
                                             headerFields:nil];

    [self.client URLProtocol:self didReceiveResponse:response 
    			  cacheStoragePolicy:NSURLCacheStorageNotAllowed];
    [self.client URLProtocol:self didLoadData:body];
}

- (NSData *)getResponseBody {
    NSString *returnJson = [NSString stringWithFormat:@"{\"result\":\"%@\"}", @"Success"];
    return [returnJson dataUsingEncoding:NSUTF8StringEncoding];
}

```

**- (void)stopLoading** is triggered when the request is cancelled by the client. This is used to close open connections, which is not the case here.

More detailed information about ``NSURLProtocol`` can be found in [the official documentation][nsurlprotocol].

#Enabling Async calls

The implementation above has a potential problem: it's not asynchronous, which means the ``UIWebView`` is frozen until the response comes back. It's important to understand that **it's not your web app the freezes, it's the ``UIWebView`` itself.** In some scenarios this can be a big problem, specially if the operation performed on the native app is likely to take some time.

Fortunately, knowing how to switch messages between native and web is enough to perfom the task asynchronously. The basic idea is to intercept the request, fire some threads in the background, respond with HTTP code ``202 (Accepted)`` and, when the background thread is done, send the results using javascript events.

In this scenario, the ``NSURLProtocol`` is responsible for returning a response indicating that the request was accepted and starting the thread that will do the actual work. It is this thread responsibilty to then execute the javascript event containing the results or any error that might have happened during the execution.

#Wrapping up

Hybrid mobile apps bring the best of both worlds, but it also comes with new problems to be solved. Switching message between completely different layers is fundamental, specially when the native side is not only a ``UIWebView`` wraping the web app. Knowing how to make these two layers communicate properly is essential to create robust hybrid mobile applications.

[nsurlprotocol]: https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSURLProtocol_Class/index.html