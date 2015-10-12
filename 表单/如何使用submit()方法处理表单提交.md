[原文地址](http://symfony.com/doc/current/cookbook/form/direct_submit.html) http://symfony.com/doc/current/cookbook/form/direct_submit.html

#如何使用submit()方法处理表单提交

通过handleRequest()(symfony2.3以前)方法处理表单提交真的非常简单,

    use Symfony\Component\HttpFoundation\Request;
    // ...

    public function newAction(Request $request)
    {
        $form = $this->createFormBuilder()
            // ...
            ->getForm();

        $form->handleRequest($request);

        if ($form->isValid()) {
            // perform some action...

            return $this->redirectToRoute('task_success');
        }

        return $this->render('AppBundle:Default:new.html.twig', array(
            'form' => $form->createView(),
        ));
    }

小提醒:
>查看更多的有关这个方法信息,请阅读[处理表单提交]()

##手动调用Form::submit()

在某些情况下,你想要更好的控制何时表单提交且传递数据给它.不是使用handRequest()方法,通过直接使用submit()方法

    use Symfony\Component\HttpFoundation\Request;
    // ...

    public function newAction(Request $request)
    {
        $form = $this->createFormBuilder()
            // ...
            ->getForm();

        if ($request->isMethod('POST')) {
            $form->submit($request->request->get($form->getName()));

            if ($form->isValid()) {
                // perform some action...

                return $this->redirectToRoute('task_success');
            }
        }

        return $this->render('AppBundle:Default:new.html.twig', array(
            'form' => $form->createView(),
        ));
    }

表单由嵌套字段组成期望在submit()中是一个数组,你也可以通过在字段上直接调用submit()来提交单个字段

    $form->get('firstName')->submit('Fabien');

##传递一个Request给Form::submit()（过时了）

在symfony2.3以前,submit()方法接收一个Request对象作为一个方便快捷的前面示例:

    use Symfony\Component\HttpFoundation\Request;
    // ...

    public function newAction(Request $request)
    {
        $form = $this->createFormBuilder()
            // ...
            ->getForm();

        if ($request->isMethod('POST')) {
            $form->submit($request);

            if ($form->isValid()) {
                // perform some action...

                return $this->redirectToRoute('task_success');
            }
        }

        return $this->render('AppBundle:Default:new.html.twig', array(
            'form' => $form->createView(),
        ));
    }

直接将Request传递给submit()方法任然能够工作,但是过时了,在symfony3.0将要删除,你需要使用的是handleRequest()方法

