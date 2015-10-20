[原文地址](http://symfony.com/doc/current/cookbook/logging/monolog_regex_based_excludes.html)

#如何配置Monolog从日志中排除掉404错误

有时候你的日志因为充满了不必要的404 http错误,例如,当攻击者扫描你的应用一些常见的应用路径(如:/phpmyadmin)
当使用fingers_crossed handler,你可以在MonologBundle配置中基于正则排除掉404错误日志

    monolog:
        handlers:
            main:
                # ...
                type: fingers_crossed
                handler: ...
                excluded_404s:
                    - ^/phpmyadmin



