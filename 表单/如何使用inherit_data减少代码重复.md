[原文地址]()

#如何使用inherit_data减少代码重复

该inherit_data表单字段选项常用于当你在不同的实体有一些重复的字段,例如,假如你又两个实体，
一个Company和一个Customer

    // src/AppBundle/Entity/Company.php
    namespace AppBundle\Entity;

    class Company
    {
        private $name;
        private $website;

        private $address;
        private $zipcode;
        private $city;
        private $country;
    }

    // src/AppBundle/Entity/Customer.php
    namespace AppBundle\Entity;

    class Customer
    {
        private $firstName;
        private $lastName;

        private $address;
        private $zipcode;
        private $city;
        private $country;
    }

正如你看到的,每个实体共享一些相同的字段: address,zipcode,city,country

开始建立两个表单实体,CompanyType和CustomerType:

    // src/AppBundle/Form/Type/CompanyType.php
    namespace AppBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;

    class CompanyType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('name', 'text')
                ->add('website', 'text');
        }
    }

    // src/AppBundle/Form/Type/CustomerType.php
    namespace AppBundle\Form\Type;

    use Symfony\Component\Form\FormBuilderInterface;
    use Symfony\Component\Form\AbstractType;

    class CustomerType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('firstName', 'text')
                ->add('lastName', 'text');
        }
    }

在两个表单中包含了除了不是重复的字段address,zipcode,city和country在两个表单中,创建一个地方的表单
称它为LocationType

    // src/AppBundle/Form/Type/LocationType.php
    namespace AppBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;
    use Symfony\Component\OptionsResolver\OptionsResolver;

    class LocationType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder
                ->add('address', 'textarea')
                ->add('zipcode', 'text')
                ->add('city', 'text')
                ->add('country', 'text');
        }

        public function configureOptions(OptionsResolver $resolver)
        {
            $resolver->setDefaults(array(
                'inherit_data' => true
            ));
        }

        public function getName()
        {
            return 'location';
        }
    }

该location表单设置了一个有趣的选项,名称为inherit_data.这个选项是让表单继承他的父表单数据
如果嵌入在company表单中.location表单能访问Company实例字段属性,如果嵌入在customer表单中,
将能访问Customer实例的字段属性.很简单,是吧?

附注:
>除了在LocationType里设置inherit_data选项.你也能够给$builder->add()传递第三个参数

最后,这部分工作是给你的两个原表单添加Location表单:

    // src/AppBundle/Form/Type/CompanyType.php
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        // ...

        $builder->add('foo', new LocationType(), array(
            'data_class' => 'AppBundle\Entity\Company'
        ));
    }

    // src/AppBundle/Form/Type/CustomerType.php
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        // ...

        $builder->add('bar', new LocationType(), array(
            'data_class' => 'AppBundle\Entity\Customer'
        ));
    }

就是这样.你已经把重复字段的定义抽取到了一个单独的location表单中,可以在需要的地方使用它

提醒:
>表单使用inherit_data选项设置不能有*_SET_DATA 事件监听器
