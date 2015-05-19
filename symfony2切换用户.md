#如何模拟一个用户
>有时候我们需要从一个用户切换到另外一个用户来进行调试程序，我们可能需要退出当前用户再登陆另外一个用户；这样做很麻烦；symfony2可以只用激活switch_user即可

	# app/config/security.yml
	security:
	    firewalls:
	        main:
	            # ...
	            switch_user: true

>我们只用在url加上一个参数即可 `http://example.com/somewhere?_switch_user=thomas` 切换回原用户`http://example.com/somewhere?_switch_user=_exit`，注意切换的用户必须有 `ROLE_ALLOWED_TO_SWITCH ` 角色，当然这个也可以设置

	# app/config/security.yml
	security:
	    firewalls:
	        main:
	            # ...
	            switch_user: { role: ROLE_ADMIN, parameter: _switch_user这个即是url参数  }