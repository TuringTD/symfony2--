[原文地址](http://symfony.com/doc/current/cookbook/logging/monolog_email.html)

#如何配置Monolog发送错误邮件

当你应用里发生了错误,Monolog能配置发送一封邮件,该配置必须嵌套一个新的handlers为了避免接收太多的邮件.
这个配置第一眼看起来很复杂,但是拆开来每个handler是相当简单的.

    # app/config/config_prod.yml
    monolog:
        handlers:
            mail:
                type:         fingers_crossed
                # 500 errors are logged at the critical level
                action_level: critical
                # to also log 400 level errors (but not 404's):
                # action_level: error
                # excluded_404s:
                #     - ^/
                handler:      buffered
            buffered:
                type:    buffer
                handler: swift
            swift:
                type:       swift_mailer
                from_email: error@example.com
                to_email:   error@example.com
                # or list of recipients
                # to_email:   [dev1@example.com, dev2@example.com, ...]
                subject:    An Error Occurred!
                level:      debug

该mail handler 是一个fingers_crossed handler意味着他仅仅在当控制级别在critical情况下触发,该ctritical级别
仅仅是HTTP 错误代码为 5xx时触发;如果级别一旦达到.该fingers_crossed handler将记录所有信息不管什么级别。该
handler的设置意味着将输出传递到buffered handler.

>小提醒:
如果你想要400级别和500级别都触发邮件,设置action_level为error而不是critical.查看上面例子代码

buffered handler 简单的保持了一个请求的所有信息,然后传递他们到嵌套handler里.如果你不使用这个handler，每条信息
将单独收邮件,这个然后传递到swif handler,这个handler实际是发送错误邮件,这个设置非常简单，发送，收件地址以及邮件主题

你可以将这些handler和其他的handler合并,这样既可以发送邮件又可以把信息记录在文件上

    # app/config/config_prod.yml
    monolog:
        handlers:
            main:
                type:         fingers_crossed
                action_level: critical
                handler:      grouped
            grouped:
                type:    group
                members: [streamed, buffered]
            streamed:
                type:  stream
                path:  "%kernel.logs_dir%/%kernel.environment%.log"
                level: debug
            buffered:
                type:    buffer
                handler: swift
            swift:
                type:       swift_mailer
                from_email: error@example.com
                to_email:   error@example.com
                subject:    An Error Occurred!
                level:      debug

这使用了group handler将信息发送给组里面的两个成员,buffered和streamed,现在日志即能写文件又能发送邮件







