表单
========

在web开发中处理表单是一个常见，具有挑战性的任务；symfony2集成了表单组件;用它来创建表单是非常容易的；在这一章节
你将从上到下学习创建一个复杂的表单;在学习过程中你将学习到表单类库的一些重要特征

小提示:
> symfony表单组件是一个独立的组件可以使用于symfony以外的项目中，更多信息;请查看[symfony组件文档](http://symfony.com/doc/current/components/form/introduction.html)

#创建一个简单的表单

假象一下需要做一个展示待办事项列表的应用；因为你需要创建或修改任务;你将需要建立一个表单;在开始以前,首先我们来先看看单个任务在Task类中的展现和数据存储的方式


    // src/AppBundle/Entity/Task.php
    namespace AppBundle\Entity;

    class Task
    {
        protected $task;
        protected $dueDate;

        public function getTask()
        {
            return $this->task;
        }

        public function setTask($task)
        {
            $this->task = $task;
        }

        public function getDueDate()
        {
            return $this->dueDate;
        }

        public function setDueDate(\DateTime $dueDate = null)
        {
            $this->dueDate = $dueDate;
        }
    }

因这个是一个普通的php对象,到现在为止，他没有在symfony或其它类库中做任何事; 它是一个非常简单普通的php对象,在你的应用程序中直接解决了一个问题(如: 在你的应用程序中代表一个任务)
当然,结束这一章节,你将能够提交一个task实例(通过html表单),验证这数据,并且持久化到数据库

##建立表单

现在你已经创建了一个task类;下一步创建一个实际渲染html表单,在symfony中,完成这部分是通过创建一个表单对象且在模板中渲染它,现在,这所有的过程可以在一个控制器中完成

    // src/AppBundle/Controller/DefaultController.php
    namespace AppBundle\Controller;

    use AppBundle\Entity\Task;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\HttpFoundation\Request;

    class DefaultController extends Controller
    {
        public function newAction(Request $request)
        {
            // create a task and give it some dummy data for this example
            $task = new Task();
            $task->setTask('Write a blog post');
            $task->setDueDate(new \DateTime('tomorrow'));

            $form = $this->createFormBuilder($task)
                ->add('task', 'text')
                ->add('dueDate', 'date')
                ->add('save', 'submit', array('label' => 'Create Task'))
                ->getForm();

            return $this->render('default/new.html.twig', array(
                'form' => $form->createView(),
            ));
        }
    }

小提示:
 >这个例子展现如何在一个控制器中建立一个表单,后来,在"[创建一个表单类](http://symfony.com/doc/current/book/forms.html#book-form-creating-form-classes)"这部分中，
 你将学习到如何独立的创建一个表单类;推荐使用这种方式,因为可重用


创建一个表单需要很少的代码因为symfony表单对象建立了一个"form builder" 这个form builder目的是允许你写一个简单的表单"食谱"且事实上他做了创建表单的一些繁重的任务;

在这个例子中你有增加两个字段 task 和 dueDate 对应ask类中的 task,dueData成员属性,你也有为他们每个赋予了不同的表单类型(text, date), 其中,除了一下其它事项外,它决定了该字段的渲染的html
标签,

最后你增加了一个自定义label的提交按钮用于吧表单提交给服务端

symfony内置了好多表单类型,在不久我门将要讨讨论([内置表单类型](http://symfony.com/doc/current/book/forms.html#book-forms-type-reference))


##渲染表单

现在你已经表单创建完成了,下一步是渲染它,这是通过一个特殊的表单"view"对象给你的模板(注意上面的例子是通过`$form->createView()`)且使用一些表单帮助函数

    {# app/Resources/views/default/new.html.twig #}
    {{ form_start(form) }}
    {{ form_widget(form) }}
    {{ form_end(form) }}


注解:
   >这个例子中假如你post提交到同一URL中,你将要学习如何改变请求的方法和提交表单的目标地址


就这样!只需要三行代码就能渲染完整的表单

`form_start(form)`

渲染表单开始标签,当使用文件上传时包括正确的enctype属性

`form_widget(form)`

渲染所有字段,其中包括字段元素本身,一个label(字段名称)和一个字段的任何验证错误信息


`form_end(form)`

渲染表单结束标签和任何尚未渲染的字段,如果你渲染每个字段自己,这个长用于渲染隐藏字段和利用自动[crsf防护](http://symfony.com/doc/current/book/forms.html#forms-csrf)

这很容易,但是他不是非常灵活, 通常, 你想要独立的渲染表单的每个字段，所以你能够控制表单的展现,你将在"在模板中渲染一个表单"的章节中学习到;


在继续前进之前，注意task输入框里面的值是如何从$task对象中获取到的（如：写一篇blog),表单的第一工作就是从对象中获取数据且转化成一种格式,适合在html形式中呈现


小提示:
>表单系统非常聪明的通过Tasks类中的getTask()和setTask()来访问受保护的task属性的值,除了属性是public,其它的必须有相应的get和set方法,到现在为止表单能够获取和设置属性的值
对于boolean型的属性,你能够使用"isser"或"hasser"方法(如  isPublished() or hasReminder() )而不是一个getter(如 getPublished() or getReminder())


##处理表单提交

表单的第二份工作转化用户提交的数据设置到一个对象中的属性; 让这种情况发生
