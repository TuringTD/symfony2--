[原文地址](http://symfony.com/doc/current/cookbook/logging/channels_handlers.html)

#如何配置Monolog日志信息写入不同文件

symfony框架组织日志信息到通道.默认,这又很多通道,包括doctrine,event,security,request等等.
该通道是打印日志信息,且可以指导不同的通道用不同的地方/文件

默认,symfony每条日志信息到了一单个文件里(不管什么通道)

>附注:
每个通道在容器(使用debug:container命令可以查看完整的服务列表)中对应一个logger服务(monolog.logger.xxx)
这些都是注入不同的服务

##切换一个通道到一个不同的Handler

现在,假如你想要记录security通道到不同的文件.可以这么做,只创建一个新的handler和配置它只记录
security channel的信息.你也可以添加到config.yml中应用于所有环境.或者只是config_prod.yml中
会在prod环境中有效

    # app/config/config.yml
    monolog:
        handlers:
            security:
                # log all messages (since debug is the lowest level)
                level:    debug
                type:     stream
                path:     "%kernel.logs_dir%/security.log"
                channels: [security]

            # an example of *not* logging security channel messages for this handler
            main:
                # ...
                # channels: ["!security"]


##YAML 规范

你可以通过多种方式来指定配置

    channels: ~    # 包括所有通道

    channels: foo  # 仅包含 "foo"
    channels: "!foo" # 包含所有,除了 "foo"

    channels: [foo, bar]   # 仅包含"foo" 和 "bar"
    channels: ["!foo", "!bar"] # 包含所有,除了 "foo" and "bar"


##创建你自己的通道
你可以一下子改变该通道 monolog日志作为一个服务,这是通过下面配置来完成或通过给你里的服务里添加
添加monolog.logger Tag且指定服务日志通道,在tag里,该logger是注入服务的是预先配置使用你
指定的通道.

##


