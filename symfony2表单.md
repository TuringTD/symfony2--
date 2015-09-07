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

表单的第二份工作转化用户提交的数据设置到一个对象中的属性; 为了实现这一点, 用户提交的数据必须要写入到表单中,在你的控制器中增加下面的代码

    use Symfony\Component\HttpFoundation\Request;

    public function newAction(Request $request)
    {
        // just setup a fresh $task object (remove the dummy data)
        $task = new Task();

        $form = $this->createFormBuilder($task)
            ->add('task', 'text')
            ->add('dueDate', 'date')
            ->add('save', 'submit', array('label' => 'Create Task'))
            ->getForm();

        $form->handleRequest($request);

        if ($form->isValid()) {
            // perform some action, such as saving the task to the database

            return $this->redirectToRoute('task_success');
        }

        // ...
    }

这个控制器遵循了一个公共的处理表单模式,有三个可能的路径

1. 当浏览器开始加载一个页面,简单的创建了和渲染一个表单. handleRequest()方法能够识别到表单是没有提交的.如果这个
表单没有提交isValid()方法是返回false的;

2. 当这个用户提交了表单,handleRequest()方法识别到且立即把提交的数据写入到$task对象中的task和dueData属性中
当这个对象经过验证.如果是无效(验证在下一节讨论),isValid()再次返回false,所以这个表单再次和验证错误信息一起渲染;

  小提示:
      >你也能够使用isSubmitted()方法来检查表单是否提交,不管提交的数据是否有效

3. 当这个用户提交表单数据是有效的,这个提交数据再次写到表单中,但是这次isValid()方法返回true.现在你有机会
在这个用户重定向一些页面(如:"感谢"或"成功"页面)前使用$task对象执行一些动作(如：持久化到数据库)

 小提示:
     >表单提交提交成功重定向;防止这个用户点击浏览器的刷新按钮重复提交数据

 如果你想要控制表单何时提交或数据传递给它,你能够使用那个submit()方法,在cookboook中阅读更多的信息



##多个按钮提交表单

当你的表单包含多个提交按钮,在你的控制器中你将想要检测点击的按钮执行合适的程序,要做到这一点,在你的表单第二个按钮
加上标题"保存和添加"

    $form = $this->createFormBuilder($task)
        ->add('task', 'text')
        ->add('dueDate', 'date')
        ->add('save', 'submit', array('label' => 'Create Task'))
        ->add('saveAndAdd', 'submit', array('label' => 'Save and Add'))
        ->getForm();

在你的控制器中,使用按钮的 isClicked()方法来检测"保存和添加"按钮是否点击;

    if ($form->isValid()) {
        // 执行一些动作，比如保存task到数据库

        $nextAction = $form->get('saveAndAdd')->isClicked()
            ? 'task_new'
            : 'task_success';

        return $this->redirectToRoute($nextAction);
    }

##表单验证

在前面章节部分，你学习了如何能够表单的数据是否有效， 在symfony中,验证是应用于底层对象(如:Task),换句话来说
这个问题不是表单是否有效,但验证$task对象是否有效是基于表单提交后的它，调用$form->isValid()方法是验证$task对象数据是否
有效的快捷方法

验证时通过给一个类添加一些规则(称呼为约束)来完成的,看到这,添加验证约束task字段不能为空和dueDate字段不能为空且必须
为有效的DateTime对象

    这里只列出yml格式；有四种方式可用；具体查看官网
    # AppBundle/Resources/config/validation.yml
    AppBundle\Entity\Task:
        properties:
            task:
                - NotBlank: ~
            dueDate:
                - NotBlank: ~
                - Type: \DateTime


这是他!如果你重新提交无效数据, 你将看到相应的错误信息打印到表单中

附注:

  >html5 验证
  >自html5出来,许多浏览器能够自然的在客户端执行一些验证.通常通过给字段渲染一个required属性来激活验证,对于
  >支持html5的浏览器,如果用户尝试提交一个空的数据则浏览器将要显示一个天然（内置的）的信息

  在生成表单的时候可以利用这些特征添加一些html属性来触发验证.客户端这边验证，然而,能够通过给form标签或formnovalidate 添加novalidate 属性来
  禁用,这个通常用于当你想要测试服务端验证的时候使用，但是这可能被浏览器阻止，例如,提交一个空白字段.

  `{{ form(form, {'attr': {'novalidate': 'novalidate'}}) }}`

验证是symfony非常强大的特征,有专门章节来讲解


##验证组

如果你的对象验证需要利用验证组，你将需要指定你的表单所使用的验证组

    $form = $this->createFormBuilder($users, array(
        'validation_groups' => array('registration'),
    ))->add(...)

如果你使用表单类(好的实践方式),你将需要添加下面的configureOptions方面

    use Symfony\Component\OptionsResolver\OptionsResolver;

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'validation_groups' => array('registration'),
        ));
    }

上面两种情况,仅registration 验证组将要用于潜在对象的验证

##禁用验证

有时候他用于完全抑制一个表单的验证，这种情况你可以设置validation_groups 选项为false;

    use Symfony\Component\OptionsResolver\OptionsResolver;

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'validation_groups' => false,
        ));
    }

注意当你做了这些的事情以后,这个表单任然会做一些基本完整性的验证,例如当你上传一个非常大的文件或提交一个不存在的字段,如果想要
抑制验证,你可以使用表单 POST_SUBMIT 事件.

##根据提交的数据决定验证组

如果你有时候需要根据先进的逻辑决定验证组(如:基于提交的数据),你能够设置validation_groups 选项为一个数组回调

    use Symfony\Component\OptionsResolver\OptionsResolver;

    // ...
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'validation_groups' => array(
                'AppBundle\Entity\Client',
                'determineValidationGroups',
            ),
        ));
    }

当表单提交后将要调用Client类中determineValidationGroups静态方法,但在执行验证前，这个表单对象通过一个参数给你这个方法(看下面这个例子)
你也可以使用闭包来完成
    use AppBundle\Entity\Client;
    use Symfony\Component\Form\FormInterface;
    use Symfony\Component\OptionsResolver\OptionsResolver;

    // ...
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'validation_groups' => function (FormInterface $form) {
                $data = $form->getData();

                if (Client::TYPE_PERSON == $data->getType()) {
                    return array('person');
                }

                return array('company');
            },
        ));
    }

使用这个validation_groups 选项覆盖默认验证组,如果你想要同时使用默认约束来验证实体，需要按照下面来调整选项

    use AppBundle\Entity\Client;
    use Symfony\Component\Form\FormInterface;
    use Symfony\Component\OptionsResolver\OptionsResolver;

    // ...
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'validation_groups' => function (FormInterface $form) {
                $data = $form->getData();

                if (Client::TYPE_PERSON == $data->getType()) {
                    return array('Default', 'person');
                }

                return array('Default', 'company');
            },
        ));
    }

你能够在validation groups章节中找到有关验证组和默认约束工作相关的更过的信息;

##根据点击按钮决定验证组

当你表单有多个提交按钮,你能够根据用户点击那个按钮来改变表单的验证组，例如,思考一下有一个向导表单 让你点击下一步前进,点击上一步
返回，还假设当点击上一步返回需要保存数据但是不需要验证。

首先你需要给表单增加两个按钮

    $form = $this->createFormBuilder($task)
        // ...
        ->add('nextStep', 'submit')
        ->add('previousStep', 'submit')
        ->getForm();

其次，我们需要为上一步按钮配置运行指定验证组，在这个例子中我们想要抑制验证,所以我们设置validation_groups选项为false;

    $form = $this->createFormBuilder($task)
        // ...
        ->add('previousStep', 'submit', array(
            'validation_groups' => false,
        ))
        ->getForm();

现在这个表单将要跳过验证约束，但是他任然会验证一些基本完整性的约束，比如检查上传文件是否太大或者你尝试在一个number字段中提交
文本数据

##建立字段类型

symfony拥有大量标准的字段类型组，覆盖了我们常遇到的表单字段和数据类型


各个字段的基本使用请查看文档，这里只是列出来

###文本字段

 >text, textarea, email, integer, money, number, password, percent, search, url

###选择字段类型

 >choice, entity, country, language, locale, timezone, currency

###日期和时间类型

 >date, datetime, time, birthday

###其它字段

 >checkbox, file, radio

###字段组

 >collection, repeated

###隐藏字段

 >hidden

###按钮

 >button, reset, submit

###基本字段

 >form

你也能够自己创建字段类型,这个主题将要在cookbook中的"[如和创建自定义字段](http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html)"章节中讨论


##字段类型选项

每个字段都有很多选项来配置它,例如,这个dueDate字段目前是渲染说三个下拉框,然而,date 字段能够配置出渲染单个文本框（用户在输入框中输入日期将作为字符串）

 `->add('dueDate', 'date', array('widget' => 'single_text'))`

每个字段类型都有很多不同的选项传递给它，这些特定的字段类型和细节可以在每种类型的文档中找到;

  附注:
  require选项

  require选项是一个常用选项,它应用于任何字段,默认该选项为true,这意味着如果字段留空,支持html5浏览器将应用客户端验证，如果你不想要
  这种行为,也可以设置require属性为false或禁用html5验证

  还需要注意的是当你设置require为true，不会导致服务端验证.换句话来说.如果提交一个空白字段值(也许是旧的浏览器或者web service)
  它将要被接受成一个有效值除非你使用symfony NotBlank或NotNull验证约束

  换句话说.这reuqire选项是"漂亮".但是在服务端也会是需要验证

  label选项

  表单字段的label可以通过设置label选项,他能应用于任何字段

    ->add('dueDate', 'date', array(
        'widget' => 'single_text',
        'label'  => 'Due Date',
    ))

   你一个字段label也可以在表单渲染模板中设置,看下面,如果你不需要label关联你的字段,你可以通过设置它的值为false来禁用它

##字段类型猜测

现在你已经给Task类添加了验证,symfony已经知道了你字段的一点信息,如果你允许它,symfony能够猜测你字段的类型且为你设定,
这个例子中,symfony能够从验证规则猜测 task字段为普通text字段和dueDate字段为date类型

    public function newAction()
    {
        $task = new Task();

        $form = $this->createFormBuilder($task)
            ->add('task')
            ->add('dueDate', null, array('widget' => 'single_text'))
            ->add('save', 'submit')
            ->getForm();
    }


当你忽略add()方法的第二个参数这"猜测"就被激活(或者通过设置为null),如果你通过一个数组作为第三个参数(上面的dueDate字段),这些选项
也应用于字段猜测

注释:
> 如果你表单使用了一个指定的验证组,当猜测你的字段类型，这字段猜测会考虑所有验证约束(包括验证组中没有用到的验证)。


###字段类型选项猜测

除了猜测字段类型,symfony也能够尝试猜测一个字段大部分选项的正确值

小提示:

>当这些选项设置了,这字段将要渲染专门的hml属性，提供一些html5客户端验证，然而，他不等同于生成了服务端约束(如果 Asset\Length)
虽然你将需要手动增加一些约束，从这些信息也能够猜测字段类型选项

required

根据验证规则(如NotBlank或NotNull)能够猜测required选项或Doctrine的metadata(这个字段 nullable),这个是非常常用，将自动匹配
验证规则作为客户端验证

max_length

如果这些字段是一些文本字段,从这些字段的验证规则(如果使用了Length或Range)能够猜测出max_length选项或
Doctrine metaData（通过这个字段的Length）

注意:
>如果你使用symfony字段猜测（通过忽略add()第二个参数或设为null）这些字段选项仅仅是猜测

如果你想要改变一个猜测字段选项的值,你能够在字段选项数组中覆盖这个选项；


    ->add('task', null, array('attr' => array('maxlength' => 4)))


##在模板中渲染表单

迄今为止,你所看到的是只是通过一行代码来渲染真个表单的,当然,你将要使用非常灵活的方式去渲染

    {# app/Resources/views/default/new.html.twig #}
    {{ form_start(form) }}
        {{ form_errors(form) }}

        {{ form_row(form.task) }}
        {{ form_row(form.dueDate) }}
    {{ form_end(form) }}

你已经知道了from_star和from_end的功能,但是其他的功能你知道吗?

`form_errors(form)`

将任何错误呈现给整个表单（在每个字段中显示字段特定错误）

`form_row(form.dueDate)`

渲染label,任何错误和html表单小物件在一个特定的元素里面,默认情况下是一个div元素

大多数工作是由from_row来完成的，渲染label,错误和表单相关元素,每个字段默认渲染是在一个div标签里，在表单主题章节中你将要学习到
from_row能够在不同层面上自定义输出

小提示:
> 你能够通过`form.vars.value`来访问当前的表单数据

    {{ form.vars.value.task }}

###手动渲染每个字段

frow_row帮助方法是非常棒的因为你能够非常快的渲染你表单的每一个字段(且每一行的标记都可以自定义),但是由于生活并不总是那么简单
你也能够手动完全渲染每一个字段,当你使用from_row最终的产品和下面是一样的;

    {{ form_start(form) }}
        {{ form_errors(form) }}

        <div>
            {{ form_label(form.task) }}
            {{ form_errors(form.task) }}
            {{ form_widget(form.task) }}
        </div>

        <div>
            {{ form_label(form.dueDate) }}
            {{ form_errors(form.dueDate) }}
            {{ form_widget(form.dueDate) }}
        </div>

        <div>
            {{ form_widget(form.save) }}
        </div>

    {{ form_end(form) }}

如果自动生成的label并不十分正确你能够明确的指定它

    {{ form_label(form.task, 'Task Description') }}

一些字段类型有一些额外的渲染选项传递给小物件,这些选项在每个类型的文档中有说明,但是公共的选项attr,允许你修改表单元素的
属性,以下将添加task_field样式类给文本字段渲染

    {{ form_widget(form.task, {'attr': {'class': 'task_field'}}) }}

如果你需要通过手动渲染字段然后访问每个字段独立的值如 id,label,下面的例子是获取id

    {{ form.task.vars.id }}

获取值使用表单字段的名称属性你必须使用全名

    {{ form.task.vars.full_name }}


###twig模板函数参考

如果你使用twig,完整的引用表单渲染方法可以在[参考手册](http://symfony.com/doc/current/reference/forms/twig_reference.html)中找到
阅读这一切有关助手和每一个可用的选项


##改变表单的提交方法

到目前为止,from_start()助手已经渲染了表单的开始标签和我们假想表单提交POST请求到一样的URL,有时候你想要改变这些参数
你可以使用几个不同的方式去做,如果你是在你的控制器中创建的表单,你能够使用setAction()和setMethod()方法

    $form = $this->createFormBuilder($task)
        ->setAction($this->generateUrl('target_route'))
        ->setMethod('GET')
        ->add('task', 'text')
        ->add('dueDate', 'date')
        ->add('save', 'submit')
        ->getForm();

注释:
> 这个实例是假设你已经创建了一个名称为target_route的路由在控制器中处理表单

在表单类中,你将学习到如何将构建表单的代码移动到一个独立的类中,当在控制器中使用一个外部表单类,你能够通过action和method作为表单
的选项

    $form = $this->createForm(new TaskType(), $task, array(
        'action' => $this->generateUrl('target_route'),
        'method' => 'GET',
    ));

最后,你能够通过form()或form_start()助手在模板中覆盖action和method

    {{ form_start(form, {'action': path('target_route'), 'method': 'GET'}) }}

注释:
> 如果你的表单方法不是GET和POST且不是PUT,PATCH 或 DELETE,symfony将要插入一个隐藏的名称为_method的字段来存储这个方法,这个
表单提交将是一个普通的POST请求,但symfony的路由是有能力检查_method参数和将要解释它作为一个 PUT, PATCH 或 DELETE的请求
阅读cookbook中"如何使用HTTP方法除了GET和POST之外的方法"来了解更过的信息


##创建表单类

正如你所看到的,一个表单的创建和使用都是直接在控制器中,然而,一个更好的实践方式建立一个单独的PHP类来分离,这样你的应用程序中任何
地方能够重复利用,创建一个新的类来安置建立task表单的逻辑


    // src/AppBundle/Form/Type/TaskType.php
    namespace AppBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;

    class TaskType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('task')
                ->add('dueDate', null, array('widget' => 'single_text'))
                ->add('save', 'submit')
            ;
        }

        public function getName()
        {
            return 'task';
        }
    }

注意:
>getName()方法返回是一个标识符,这种表单类型,这个标识符在你应用中必须唯一,除非你想要覆盖一个内置的类型,在你的应用程序
中它应该不同于默认的symfony类型和你安装的第三方bundle中的任何定义的类型,考虑给你的类型加一app_前缀避免标识碰撞

这个新类中包含了所有的创建任务表单的所需的用法,在控制器中能够快速的建立一个表单对象

    // src/AppBundle/Controller/DefaultController.php

    // add this new use statement at the top of the class
    use AppBundle\Form\Type\TaskType;

    public function newAction()
    {
        $task = ...;
        $form = $this->createForm(new TaskType(), $task);

        // ...
    }

把这个表单的逻辑放置到自己的类中这意味着在你的项目中任何地方你能够很简单的重复利用,这是最好的创建表单方式,但最终的选择取决于你

附注:

####设置data_class

每个表单需要知道潜在数据类的名称(如 AppBundle\Entity\Task),通常是通过传递给createForm方法的第二参数的对象去猜测(如$task)
随后,当你开始嵌入表单,这将不再足够,所以,虽然并不总是必要,给你表单类型类增加明确指定的data_class选项值他通用是一个好的主意

    use Symfony\Component\OptionsResolver\OptionsResolver;

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\Task',
        ));
    }

当给表单映射一个对象时,他所有的字段都将要映射,任何字段在表单映射对象中不存在将要引起一个异常的抛出

在你表单需要额外字段的情况下(如:有一个是否同意条款的checkbox框)他将不会映射到潜在对象,你需要设置mapped选项为false

    use Symfony\Component\Form\FormBuilderInterface;

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('task')
            ->add('dueDate', null, array('mapped' => false))
            ->add('save', 'submit')
        ;
    }

另外,任何表单的字段在提交的数据中不包括将要明确的设置为null

在控制器中能够访问字段的值

    $form->get('dueDate')->getData();

此外,未映射的字段数据也能够直接修改

    $form->get('dueDate')->setData(new \DateTime());

###定义表单为服务

定义的表单类型为一个服务是一种非常好的实践方式且在你应用程序中使用它真的非常简单

服务和服务容器的处理将要在这本书中稍后讲解,阅读那章节后这些事情将更清晰

    # src/AppBundle/Resources/config/services.yml
    services:
        app.form.type.task:
            class: AppBundle\Form\Type\TaskType
            tags:
                - { name: form.type, alias: task }

就是它！现在你可以在控制器中使用它

    // src/AppBundle/Controller/DefaultController.php
     // ...

     public function newAction()
     {
         $task = ...;
         $form = $this->createForm('task', $task);

         // ...
     }

甚至在其它的表单中以内部表单类型来使用

    // src/AppBundle/Form/Type/ListType.php
    // ...

    class ListType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            // ...

            $builder->add('someTask', 'task');
        }
    }

阅读"创建你字段类型作为服务"了解更多信息

##表单和Doctrine

一个表单的目的是从一个对象(如 Task)数据转化到html表单和当用户提交表单后又转化成原对象,因此,持久化Task数据到数据库主题和
表单主题完全不相关,但是你配置了task类通过doctrine来持久化(例如你给他增加metadata映射),当表单是有效,能够完成提交后然后
持久化

    if ($form->isValid()) {
        $em = $this->getDoctrine()->getManager();
        $em->persist($task);
        $em->flush();

        return $this->redirectToRoute('task_success');
    }

如果,处于某种原因,你不能访问你的原始$task对象,你能从表单获取它

    $task = $form->getData();

更多的信息产看doctrine orm 相关章节

当表单提交后关键的事是理解它,提交的数据会立即转化成潜在对象,如果你想要持久化数据,你只需要持久化对象本身(已经包含提交后的数据)

##嵌入表单

通常你创建一个表单会从多个不同对象中包含字段,例如,一个注册表单也许包含数据属于一个user对象以及多个address对象,幸运的是
在表单组件中这是非常简单和自然

###嵌入单个对象

假想一下每个Task属于一个简单Category对象,首先,当然,创建一个Category对象

    // src/AppBundle/Entity/Category.php
    namespace AppBundle\Entity;

    use Symfony\Component\Validator\Constraints as Assert;

    class Category
    {
        /**
         * @Assert\NotBlank()
         */
        public $name;
    }

其次在Task类中增加一个新的category属性

    class Task
    {
        // ...

        /**
         * @Assert\Type(type="AppBundle\Entity\Category")
         * @Assert\Valid()
         */
        protected $category;

        // ...

        public function getCategory()
        {
            return $this->category;
        }

        public function setCategory(Category $category = null)
        {
            $this->category = $category;
        }
    }

小提示:
>已经给category属性添加了Vaild约束,级联验证相应实体,如果你忽略了这约束,子实体将不会验证

现在你的应用程序已经更新了以及反映新的需求,创建一个表单,这个category对象能够由用户来修改

    // src/AppBundle/Form/Type/CategoryType.php
    namespace AppBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;
    use Symfony\Component\OptionsResolver\OptionsResolver;

    class CategoryType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder->add('name');
        }

        public function configureOptions(OptionsResolver $resolver)
        {
            $resolver->setDefaults(array(
                'data_class' => 'AppBundle\Entity\Category',
            ));
        }

        public function getName()
        {
            return 'category';
        }
    }

最终的目的是允许一个Task的Category修改在正确的范围内,要实现这一点,给TaskType对象添加一个category字段,这个字段类型是
一个CategoryType类的实例

    use Symfony\Component\Form\FormBuilderInterface;

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        // ...

        $builder->add('category', new CategoryType());
    }

属于CategoryType的字段现在可以与TaskType类渲染

渲染category字段与Task对象原字段的方式一样

    {# ... #}

    <h3>Category</h3>
    <div class="category">
        {{ form_row(form.category.name) }}
    </div>

    {# ... #}

当用户提交表单,提交的category字段数据将用于构造一个Category实例,然后设置在Task实例的category字段上

Category实例可以通过`$task->getCategory()`来自然的访问且能持久化到数据库或随便使用它

###嵌入一个集合表单

在你的一个表单里你也可以嵌入一个集合表单(假想一下一个categor表单在多个Product子表单中),这是通过collection字段类型来完成

更多的信息查看"如何嵌入一个集合表单"cookbokk章节和collection字段类型参考

##表单主题

表单的每一部分渲染都能够自定义,你可以自由的更改每个表单"row"渲染,改变用于渲染错误的标签甚至定制textarea标记如何渲染,
没有什么不对且在不同的地方用不同的自定义

symfony采用模板来渲染表单的每一部分比如label标签,input标签,错误信息和其他的一切

在twig中,每个表单片是通过一个twig block来描述的,自定义任何部分的渲染,你只需要覆盖恰当的block;

在php中,每个表单片段是通过一个独立模板文件来渲染,自定义任何部分的渲染,你需要创建一个新的文件覆盖已存在的模板文件

要理解这是如何工作的,自定义form_row 片段且给每行div元素添加一个class属性,要做到这一点,创建一个新的模板文件来存取新的标记

    {# app/Resources/views/form/fields.html.twig #}
    {% block form_row %}
    {% spaceless %}
        <div class="form_row">
            {{ form_label(form) }}
            {{ form_errors(form) }}
            {{ form_widget(form) }}
        </div>
    {% endspaceless %}
    {% endblock form_row %}

form_row片段是用于大多数字段通过form_row方法来渲染,告诉表单组件你将要使用上面新定义的form_row片段,
在渲染表单的模板顶部增加以下面

    {# app/Resources/views/default/new.html.twig #}
    {% form_theme form 'form/fields.html.twig' %}

    {# 如果你使用多个模板主题 #}
    {% form_theme form 'form/fields.html.twig' 'form/fields2.html.twig' %}

form_theme标签(在Twig里)导入定义片段的模板且当表单渲染的时候使用它们.换句话来说,当form_row在模板中被调用后,他将要使用
自定的form_row block(而不是symfony内部的自带的form_row block)

你自定义的主题不是覆盖所有的blocks,当渲染一个没有被自定义覆盖的block时,主题引擎将要返回到全局主题(定义在bundle级别)

如果多个自定义主题提供了它们将要自上而下搜索到全局主题

自定义表单任何部分,你只需要覆盖合适的片段,确切的知道哪些块或文件重写是一节的主题

更广泛的信息,查看"如何自定义表单渲染"

##表单片段的命名

在symfony中,表单的每部分渲染--html表单元素,错误,label,等等都定义在一个基本主题中,在twig中这是一个block的集合在PHP中这是一个集合模板文件

在twig中,每个block需要定义在一个模板文件中(如 form_div_layout.html.twig),在这个文件里,你可以看到每个block需要渲染一个表单和每个默认的
字段类型

在php中,片段是独立的模板文件,默认是位于框架`Resources/views/Form`目录下

每个片段的名称都遵循相同基本规则和分成两块,通过一个下划线来分离,看一下下面几个例子

* form_row 用于通过form_row来渲染大部分字段
* textarea_widget 用于通过form_widget 渲染一个textarea的字段类型
* form_errors 用于为一个字段通过from_errors来渲染错误

每个片段遵循同样的规则: type_part type部分对应渲染字段的类型(如textarea, checkbox, date等)而part部分对应渲染部分(如label, widget, errors等)
默认一个表单这有四个可能的部分可能渲染

| label | (如 form_label) | 渲染字段的label
| widget | (如 form_widget) | 渲染字段的html描述
| errors | (如 form_errors) | 渲染字段的错误
| row | (如form_row) | 渲染字段整个行(label,widget,errors)

附注:

>这其实还有2个其他的部分 rows 和 reset 单你应该几乎不需要担心覆盖他们

通过了解字段类型(如textarea)和你想要自定义部分(如widget),你只需要构造片段名称覆盖掉原来的(如textarea_widget)

###模板片段继承

在某些情况下,你想自定义一个不存在的片段,例如,在symfony提供的默认主题中不存在textarea_errors片段,所以如何渲染一个textarea字段类型的错误呢?

这个回答是:通过form_errors 片段,当symfony渲染一个textarea类型的错误时,它看起来首先会去找textarea_errors片段在返回到form_errors,每个字段类型
都有一个父类型(textare,text他们的父类型是form)且如果基本片段不存在symfony将会是用父类的片段

所以，只需要覆盖textarea字段错误,复制form_errors片段重命名为textarea_errors 且 自定义它,如果覆盖所有字段的错误渲染,直接复制和自定义form_errors片段;

小提示:

>在每个字段类型的 "表单类型参考"文档中可查询到每个字段的父类是否有效

###全局表单主题

在上面的例子中,你使用form_theme助手(在twig中)导入自定表单片段来调整表单,你也可以告诉symfony导入自定表单从整个项目中

####Twig

在所有的模板中从早期创建的ields.html.twig模板中动包含自定义block,修改你的应用配置文件

    # app/config/config.yml
    twig:
        form_themes:
            - 'form/fields.html.twig'

现在任何在 fields.html.twig 模板中定义的表单输出都应用于全局

附注:

#####在单个twig文件里自定义所有表单输出

在twig中,你也可以自定义表单block在你需要自定义的模板文件里

    {% extends 'base.html.twig' %}

    {# import "_self" as the form theme #}
    {% form_theme form _self %}

    {# 制作自定义表单输出 #}
    {% block form_row %}
        {# custom field row output #}
    {% endblock form_row %}

    {% block content %}
        {# ... #}

        {{ form_row(form.task) }}
    {% endblock %}

`{% form_theme form _self %}`标签允许表单block直接使用在模板中定义的blok,使用这个方法快速的制作表单自定义输出将只需在一个模板里

`{% form_theme form _self %}` 功能 只能 在当前模板是继承其他模板的时候管用.如果你的模板没有这样做,你必须把form_theme指向一个单独的文件

####PHP

在所有的模板中从早期创建的ields.html.twig模板中动包含自定义block,修改你的应用配置文件

    # app/config/config.yml
    framework:
        templating:
            form:
                resources:
                    - 'Form'
    # ...

现在任何在 fields.html.twig 模板中定义的表单输出都应用于全局

###CRSF防护

CSRF 或[伪造请求站点](http://en.wikipedia.org/wiki/Cross-site_request_forgery)是一个恶意的用户尝试让一个你的合法用户在不知情的情况下
提交了一个你不应该提交的数据,幸运的是CSRF通过在你表里增加一个CSRF token 来阻止攻击

好消息是,默认情况下syfmony自动嵌入和验证SRF tokens ,这意味着你不需要做任何事就可以利用CSRF来防护,实际上在这一节中所利用到的表单都使用了CSRF防护

CSRF的工作原理是在你的表单增加了一个隐藏域，默认名称为_token,包含了一个只有你和你的用户知道的值,确保这是用户--不是其他实体--提交的数据
Symfony自动验证这个token的准确性

_token是一个隐藏字段且在form_end() 的时候自动在你的模板里渲染,确保所有没有渲染的字段一起输出

CRSF也一可以在一个表单中自定义,例如

    use Symfony\Component\OptionsResolver\OptionsResolver;

    class TaskType extends AbstractType
    {
        // ...

        public function configureOptions(OptionsResolver $resolver)
        {
            $resolver->setDefaults(array(
                'data_class'      => 'AppBundle\Entity\Task',
                'csrf_protection' => true,
                'csrf_field_name' => '_token',
                // a unique key to help generate the secret token
                'intention'       => 'task_item',
            ));
        }

        // ...
    }

设置csrf_protection 选项为false禁用CRSF,也可以在你项目中全局的设定,更过的信息参考表单配置相关章节

附注：
> intention选项是可选,但是给每个表单配置不同的值极大的增加生成token的安全性

CRSF意味着每个用户的值是不同的,如果尝试缓存页面包括这种保护你就需要十分小心了,更多的信息查看缓存包含CRSF防护表单页面；

##不用一个类来使用表单

在大多数情况下,一个表单捆绑一个对象且表单获取和存储每个字段的值都对应这个对象中的程序属性,在这一节你所看到的完全都是在Task类里,

有时候,你也许想要不要一个类来使用表单且获取提交数据返回一个数组,这个实际上非常简单

    //确保你导入了Request类的命令空间
    use Symfony\Component\HttpFoundation\Request;
    // ...

    public function contactAction(Request $request)
    {
        $defaultData = array('message' => 'Type your message here');
        $form = $this->createFormBuilder($defaultData)
            ->add('name', 'text')
            ->add('email', 'email')
            ->add('message', 'textarea')
            ->add('send', 'submit')
            ->getForm();

        $form->handleRequest($request);

        if ($form->isValid()) {
            // data 是一个数组,键为 "name", "email", and "message"
            $data = $form->getData();
        }

        //渲染表单....
    }

默认,一个表单实际认为你想要通过一个数组的数据来工作而不是一个对象，这有两种方式能够完全地改变这一行为和绑定一个对象到表单上

1. 创建一个表单的时候传递一个对象(作为createFormBuilder的第一个参数 和 createForm的第二个参数);
2. 给你的表单申明data_class选项

如果你不这么做,表单返回数据将会是一个数组,在这个例子中,因为$defaultData 不是一个对象(也没有设置data_class选项),`$form->getData() `
最终返回一个数组

小提示:

>你也能够直接通过request对象获取POST值

`$request->request->get('name');`

建议,在大多数情况下使用`getData()`方法是一个好的选择,因为它返回的数据(通常是对象)是竟要表单组件转化后的


##增加验证

现在唯一缺少的部分是验证,通常,当你调用`$form->isValid()`,通过读取类应用的约束来验证这个对象,如果你的表单映射的是一个对象(如:你使用data_class
选项或传递一个对象到表单)这差不多是你想用的方法,查看验证章节获取更详细的信息；

但是如果你的表单映射的不是一个对象且反倒你想要接收提交的数据是一个简单的数组,你如和在你表单中增加约束呢?

这个答案是设置约束你自己并将它们附加到各个字段,整体的验证方法在验证章节会讲解的多一些,这儿只是hi一个简短的例子

    use Symfony\Component\Validator\Constraints\Length;
    use Symfony\Component\Validator\Constraints\NotBlank;

    $builder
       ->add('firstName', 'text', array(
           'constraints' => new Length(array('min' => 3)),
       ))
       ->add('lastName', 'text', array(
           'constraints' => array(
               new NotBlank(),
               new Length(array('min' => 3)),
           ),
       ))
    ;

小提示:
>如果你用到了验证组,无论是当你创建表单时还是添加约束设置正确的组时你都需要引用Default组

`new NotBlank(array('groups' => array('create', 'update'))`

##最终的思考

你现在知道了为了给应用程序创建建立复杂功能的表单创建block是必要的,当我们创建一个表单,记住表单的第一个任务是把一个对象(Task)转化成html 表单
以便用户修改它的数据,第二个任务就是把用户提交的数据重新应用到对象上;

还有很多的表单强大功能需要学习,比如,如何在Doctrine里上传文件,如何创建一个可以添加动态数量的子表单的表单(如 提交前你可以通过js添加更多的字段)
在cookbook中查看这些主题，一定要依赖于字段类型的参考文档,其中包括了如何使用每个字段选项的例子;

