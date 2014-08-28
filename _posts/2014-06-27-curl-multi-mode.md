---
title: 记一次重构：使用 curl 异步调用接口
layout: post
tags: php 编程实践
category: 编程实践
comments: true
---
{% include JB/setup %}

##优化目标

在我现在所在的产品线中 http 接口被大量使用，用来获取各种开放数据，可以说 http 调用在代码中随处可见。比如一个访问最频繁的页面，一次请求将会产生 __7~8__ 次 http 调用。虽然每个接口都非常的快，但 8 次累加起来的消耗还是相当的可观，所以我最近的优化工作主要是：

> 并行调用各 http 请求，以缩短脚本的运行时间。

##重构起因
实际上，将请求并行化并不十分困难，使用 curl 提供的 __multi*__ 方法族就可以实现，网上有很多类似的文章来介绍其使用方法，大致的思想是把多个 curl 句柄放入一个 curl multi_handler 中，以 nonblocking 的方式执行，然后使用内核提供的 IO 复用机制（select/epoll等）进行事件查询，当有 response 返回时处理结果。很明显这是一个异步的过程，而在我们现在用的 php 框架中对于 http 调用则是同步的，建立连接、发送请求、接受响应和处理结果都是串行完成的，这些操作都被封装到一系列类中，使得上层只用一行代码就可以完成 API 的调用并获得 __返回结果__ ，例如:

	$userInfo = $this->apiProxy->general->getUserInfo($uid); //调用 api
	echo $userInfo['rst']['username']; //使用结果

上面的返回值 `userInfo`是个 __数组__ ，包含了调用方感兴趣的所有数据。虽然只有 2 行，但是框架默默帮你做了大量的工作，其中包括：建立连接、发送请求、接受结果，检查状态 、处理错误、格式化输出、写缓存等等，而且具体到每个 api 对于返回值的处理还有不同的逻辑。如何用一种优雅的方式对现有框架进行重构，既能符合要求，又能保证改动量最小，成为了现在最重要的问题。

##Lazy evaluation

当我开始接手这项工作的时候，脑海中想到的第一个对应思想就是 __Lazy evaluation__ （ __缓式求值__ ），维基百科上对于 [缓式求值](https://en.wikipedia.org/wiki/Lazy_evaluation) 的定义是：

> In programming language theory, lazy evaluation, or call-by-need[1] is an evaluation strategy which delays the evaluation of an expression until its value is needed (non-strict evaluation) and which also avoids repeated evaluations (sharing).

Lazy evaluation 是编程语言设计领域中的一个 _表达式求值_ 策略，它延缓对表达式的求值直到你需要它的时候。看上去 lazy evaluation 好像和我们的问题挨不上边，而且 php 也不支持 lazy evaluation，不过仔细想一下，如果我们能把对 http 请求的 _后续操作_ 延缓到对 _返回结果的使用_ 时，就可以用一种 优雅 的实现来使框架支持并行执行，而且对于 controller 层的改动也非常的小。

具体点说就是在进行 api 调用的时候，不再返回结果数组，而是返回一个 _句柄_，这个句柄标识了一个被提交到后台的请求，它被加入到 curl multi_handler 中，你不再关心它，由 curl 替你完成，你的代码可以继续往下执行，去完成其他的业务逻辑。而当我们需要这个结果时，检查这个句柄是否已经完成，如果已经完成则执行上面 __接受结果__ 之后的所有操作，返回结果。那么上面的代码重构后变成：

	$userInfo = $this->apiProxy->async_general->getUserInfo($uid); //使用 curl 异步调用
	//
	//执行其他的业务逻辑......
	//
	echo $userInfo['rst']['username']; //检查句柄是否已经完成，返回结果

async_ 前缀表示使用异步来调用 api 。上面代码中，在调用 api 和使用结果之间的时间都留给 curl 去连接服务器、发送请求、获取结果到 socket 输入buffer等等，就可以达到并行操作节省时间的效果。

##ArrayObject

显然接口调用的返回值必须是个 __object__ 而不能再是个 __array__ ，因为数组的可操作性有限，不能执行逻辑， __object__ 则提供了更大的灵活性，但在原有代码中 array 已经被大量应用，把它们逐个改为 object 是很不现实的，那么 object 是否可以像数组一样被使用呢？经过一番搜索，我发现 php 里还真有这样的东西，它就是 SPL(Standard PHP Library) 提供的 [__ArrayObject__](http://jp1.php.net/manual/en/class.arrayobject.php) 类，这个类的介绍简单明了：

> This class allows objects to work as arrays.

正好是我们需要的。

ArrayObject 主要通过下面 4 个方法提供对数组的支持（实际上它是通过实现 [__ArrayAccess__](http://jp1.php.net/manual/en/class.arrayaccess.php)接口来实现的）：

	public bool offsetExists ( mixed $index )
	public mixed offsetGet ( mixed $index )
	public void offsetSet ( mixed $index , mixed $newval )
	public void offsetUnset ( mixed $index )

有了这个类的帮助，我们的方案就明确了，思路就是：返回的 object 继承于 ArrayObject ，并覆盖这 4 个方法，在覆盖的方法内检查 http 请求是否完成并获取结果；而 controller 层对于结果的使用几乎不用改变，仍然按照数组方式使用。

我们把返回的句柄类命名为 __AsyncHandler__ ，它的定义为：

	class AsyncHandler extends ArrayObject

##事件回调

到目前为止，一切都非常的顺利，但是还有一个重要的问题没有解决，那就是对 http response 的处理，就像前面所说的，原有的串行方法，直接返回处理过的结果，而现在只返回一个对象，结果还不知道什么时候能取到呢，这些处理代码显然应该等到 http response 确定返回的时候才能执行，这时就要使用回调来实现了，通过对代码的分析，发现有主要 3 处对结果进行处理的代码，一处在 http response 返回时，此处做了 http 状态值的检查、日志记录等基本操作，这部分是公共的代码；另外一处在接口自身的函数内，做了接口特有的处理，这部分是每个接口一份；最后一处是在把结果返回给调用方之前，对结果做格式化，保存缓存等，这部分也是公共的代码。通过总结出类型，我们对如何修改就胸有成竹了：

首先，在 AsyncHandler 的中定义 3 个回调函数：

	//Callback functions
	protected $onRecvResponse = NULL;
	protected $recvCtx = NULL;

	protected $onBusiness = NULL;
	protected $busCtx = NULL;
	
	protected $onAPIReturn = NULL;
	protected $apiCtx = NULL;

onXXX是回调函数，xxxCtx 是回调函数的上下文信息，它是个数组，保存了回调函数执行过程中需要的变量。

在上述 3 处对 http response 进行处理的地方，把原有的代码封装成一个匿名函数，在异步 curl 模式下，把这个匿名函数和相关上下文传入到 AsyncHandler 中；如果是同步curl模式，就直接执行这个匿名函数（和原来一样）。下面以一处代码为例：

	$c['var'] = $varname; //相关上下文，就是回调函数里面用到的一些变量
	$onApiReturn = function($response, $c) {
		//
		//response 就是 api 返回的结果字符串
		//对结果进行处理，就不详细列出了
		//.......
		//
		return $response; //这里把结果返回
	};
	//this->response 异步模式下就是 AsyncHandler ，同步模式下就是结果数组。
	if(ApiProxy::$async && is_object($this->response)) {
		$this->response->setOnAPIReturn($onApiReturn, $c);
	}
	else {
		$this->response = $onApiReturn($this->response, $c);//如果是同步，直接执行处理函数
	}

在第一处处理的位置，创建 AsyncHandler 对象：

	$c['var'] = $varname; //相关上下文，就是回调函数里面用到的一些变量
	if(ApiProxy::$async) {
		$mhttp = new curl_http(); 	//curl_http 类是对 curl 基本功能的封装。
		$mhttp->setConfig($config);
		$mhttp->setUrl($url);
		$mhttp->setData($queryData);
		$mhttp->prepareRequest($reqMethod, false);

		$asynchandler = new AsyncHandler($mhttp);
		$asynchandler->setOnRecvResponse($onRecvResponse, $c);
		async_http::in()->add($asynchandler);   //async_http 是处理 curl multi 相关操作的单例
		return $asynchandler;
	}
	return $onRecvResponse(self::$http, $result, $c); //如果是同步，直接执行处理函数

mhttp 是对 curl 功能的封装，它保存 http 链接的基本信息，创建完后它被加到了 AsyncHandler 的实例中，以备后续使用，而 AsyncHandler 的实例被加到了 async_http 中，后者会把 AsyncHandler 的 curl 对象加入到 curl_multi_add_handle 方法中。

##获取结果

最后，我们看一下获取结果的流程，先看一下 AsyncHandler 的 offsetGet 方法：

	public function offsetGet($index) {
		if(!$this->finished) async_http::in()->wait((string)$this);
		return parent::offsetGet($index);
	}

首先判断是否已经拿到结果了，如果已经完成了，则调用父类的方法返回结果中对应的 index；如果没有则调用 async_http 的 wait 方法取结果：

	public function wait($ah = false) {
		$fin = false;
		do {
			do {
				$mrc = curl_multi_exec($this->mh, $this->active);
			} while($mrc == CURLM_CALL_MULTI_PERFORM);

			while ($done = curl_multi_info_read($this->mh)) {
				$asynchandler = $this->map[(string)$done['handle']];

				$asynchandler->RequestCompeleted($done['result']);

				if((string)$asynchandler === $ah) {
					$fin = true;
				}
				curl_multi_remove_handle($this->mh, $done['handle']);
			}
			if($fin)
				break;	//读完了当前能够读取的所有数据，包括想要的数据
			
			if($this->active > 0) {
				curl_multi_select($this->mh, 0.1);
			}
		} while($this->active > 0);
	}

wait 方法不断的询问 curl ：请求是否已经完成？当有结果返回时，调用对应 AsyncHandler 的 RequestCompeleted 方法：

	public function RequestCompeleted() {
		$this->_http->processRequest();
		$data = curl_multi_getcontent($this->getHttp());
		
		if(is_callable($this->onRecvResponse)) {
			$data = call_user_func($this->onRecvResponse, $this->_http, $data, $this->recvCtx);
		}
		if(is_callable($this->onBusiness)) {
			$data = call_user_func($this->onBusiness, $data, $this->busCtx);
		}
		if(is_callable($this->onAPIReturn)) {
			$data = call_user_func($this->onAPIReturn, $data, $this->apiCtx);
		}
		$this->exchangeArray($data);
		$this->finished = true;
	}

RequestCompeleted 方法获取返回结果，调用回调函数，保存结果，并置标志位。

到此我们就实现了几乎所有的异步模式功能，而且对于原来的同步模式代码改动很小，并复用了原来的大部分代码，没有冗余。代码量不大，除了新增的 2 个文件，修改部分主要是把原有的代码块用匿名函数包起来，所以也比较好测试，bug 也不多。

##使用提示

对于希望改成并行调用的多个 api ，调用方只要给它们加上 async_ 前缀，并把它们放在一起，就可以并行调用了。最后还有一点，AsyncHandler 只是个 object，虽然可以作为数组一样使用，但是对于 `is_array` 还是会返回 false 的。

##总结

本文提供了一种在已有框架下增加异步模式的设计方法，对于想要引入并行调用的朋友，希望这种方法能带来一些帮助。

##代码

AsyncHandler.php 和 async_http.class.php 的[代码](https://gist.github.com/zhujun1980/ae0d077cd7c0f3c304a3)，curl_http 类就是对 curl 功能的封装，就不再上传了，其他的代码因为和业务参考意义不大，而且也不方便上传。