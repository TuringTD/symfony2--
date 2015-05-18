#优化开发环境
>当你的symfony2项目在你本地机器上运行，你需要使用dev环境,优化这个环境有两个目的

1. 当程序发生错误，给开发者正确的反馈信息(debug 工具条，漂亮的异常页面...)
2. 当项目部署到生产环境中尽可能避免错误的发生

## 禁用bootrsrap类缓存文件
>在生产环境中为了尽可能的快，symfony创建了一个包含 每次请求需要的所有的类 的一个php缓存文件，这样做可能不利于我们调试，这篇文章就是教我怎么去调整缓存机制

	// $loader = require_once __DIR__.'/../app/bootstrap.php.cache';注释加载缓存文件
	$loader = require_once __DIR__.'/../app/autoload.php';//让其加载这个文件
	require_once __DIR__.'/../app/AppKernel.php';
	
	$kernel = new AppKernel('dev', true);
	// $kernel->loadClassCache(); //注释
	$request = Request::createFromGlobals();

