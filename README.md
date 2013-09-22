#Getting Started
```composer create-project symfony/framework-standard-edition phpmelb 2.3.4```

```cd phpmelb```

```app/console server:run``` (PHP >= 5.4)

Go to [http://localhost:8000](http://localhost:8000) and poke around the demo pages

##Remove the demo controller and configuration
```rm -R src/Acme```

**remove**

      $bundles[] = new Acme\DemoBundle\AcmeDemoBundle();

from ```app/AppKernel.php```

**remove**

    # AcmeDemoBundle routes (to be removed)
  _acme_demo:
      resource: "@AcmeDemoBundle/Resources/config/routing.yml"

from ```app/config/routing_dev.yml```

##Create our own bundle
```app/console generate:bundle```


|Option                                               |Value                              |
|-----------------------------------------------------|-----------------------------------|
|Bundle namespace                                     |```PhpMelb/BlogBundle```           |
|Bundle name                                          |```PhpMelbBlogBundle``` (*default*)|
|Target directory                                     |**default** (just press enter)     |
|Configuration format                                 |annotation                         |
|Do you want to generate the whole directory structure|yes                                |
|Do you confirm generation                            |yes (*default*)                    |
|Confirm automatic update of your Kernel              |yes (*default*)                    |
|Confirm automatic update of the Routing              |yes (*default*)                    |

Now have a look around ```src/PhpMelb/BlogBundle```

##Create a Post entity for Doctrine ORM
###Use the command line generation tool
```app/console doctrine:generate:entity```

|Option                                               |Value                              |Field type|Field Length|
|-----------------------------------------------------|-----------------------------------|
|The Entity shortcut name                             |```PhpMelbBlogBundle:Post```       |
|Configuration format                                 |annotation  (*default*)            |
|Fields                                               |title|string|255                   |
|                                                     |body |text  |*n/a*                 |
|Do you want to generate an empty repository class    |yes                                |
|Do you confirm generation                            |yes (*default*)                    |

Take a look at ```src/PhpMelb/BlogBundle/Entity/Post.php```

###Generate basic CRUD controllers, forms and templates
```app/console generate:doctrine:crud```

|Option                                               |Value                              |
|-----------------------------------------------------|-----------------------------------|
|The Entity shortcut name                             |```PhpMelbBlogBundle:Post```       |
|Do you want to generate the "write" actions          |yes                                |
|Configuration format                                 |annotation (*default*)             |
|Routes prefix                                        |/post (*default*)                  |
|Do you confirm generation                            |yes (*default*)                    |

Visit localhost:8000/post

*Oops we have an error, let's fix it*


###Create the database and table
```app/console doctrine:database:create```

####Check the SQL doctrine wants to execute then run it
```app/console doctrine:schema:update --dump-sql```

then

```app/console doctrine:schema:update --force```
####Let's make it less ugly
Replace with the following

  <!-- app/Resources/views/base.html.twig -->
  <!DOCTYPE html>
  <html>
      <head>
          <meta charset="UTF-8" />
          <meta name="viewport" content="width=device-width, initial-scale=1.0">
          <title>{% block title %}Symfony Melbourne!{% endblock %}</title>
          <link rel="icon" type="image/x-icon" href="{{ asset('favicon.ico') }}" />
          <link href="//netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap.min.css" rel="stylesheet">
          {#
          {% stylesheets
              '@FOSCommentBundle/Resources/assets/css/comments.css'
          %}
          <link rel="stylesheet" href="{{ asset_url }}" type="text/css" />
          {% endstylesheets %}
          #}

          <script src="http://code.jquery.com/jquery-1.10.2.min.js"></script>
          <script src="//netdna.bootstrapcdn.com/bootstrap/3.0.0/js/bootstrap.min.js"></script>
      </head>
      <body>
          <h1> Tom's PHP Melbourne Symfony Blog Demo</h1>
          {% block body %}{% endblock %}
          {% block javascripts %}{% endblock %}
      </body>
  </html>


Now test creating Posts, editing them and deleting

##Users
### Add FOSUserBundle
```composer require friendsofsymfony/user-bundle ~2.0@dev```

    <?php
    // app/AppKernel.php

    public function registerBundles()
    {
        $bundles = array(
            // ...
            new FOS\UserBundle\FOSUserBundle(),
        );
    }

###Generate the entity
Now create our user entity by extending ```FOS\UserBundle\Model\User```

```app/console doctrine:generate:entity```

|Option                                               |Value                              |Field type|Field Length|
|-----------------------------------------------------|-----------------------------------|
|The Entity shortcut name                             |```PhpMelbBlogBundle:User```       |
|Configuration format                                 |annotation (*default*)             |
|Fields                                               |phone|string|255                   |
|Do you want to generate an empty repository class    |yes                                |
|Do you confirm generation                            |yes (*default*)                    |


###Manually edit entity

add

``` use FOS\UserBundle\Model\User as BaseUser;```

change

``` class User extends BaseUser```

_change visibility of ```$id``` to ```protected``` and remove getter_

###Side note
Symfony is fucking amazing, DI rocks.
This bundle supports MongoDB and CouchDB with ```doctrine/odm```  propel (if you prefer a different flavour of PHP ORM)

###Update database

```app/console doctrine:schema:update --force```
###Configure ```security.yml```
    # app/config/security.yml
  security:
      encoders:
          FOS\UserBundle\Model\UserInterface: sha512

      role_hierarchy:
          ROLE_ADMIN:       ROLE_USER
          ROLE_SUPER_ADMIN: ROLE_ADMIN

      providers:
          fos_userbundle:
              id: fos_user.user_provider.username_email

      firewalls:
          dev:
              pattern:  ^/(_(profiler|wdt)|css|images|js)/
              security: false

          main:
              pattern: ^/
              form_login:
                  provider: fos_userbundle
                  csrf_provider: form.csrf_provider
              logout:       true
              anonymous:    true

      access_control:
          - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
          - { path: ^/register, role: IS_AUTHENTICATED_ANONYMOUSLY }
          - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
          - { path: ^/admin/, role: ROLE_ADMIN }

###[Update ```config.yml```](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/index.md#step-5-configure-the-fosuserbundle)
    # app/config/config.yml
    fos_user:
        db_driver: orm
        firewall_name: main
      user_class: PhpMelb\BlogBundle\Entity\User

__Uncomment translation config to enable translation component__

        translator:      { fallback: %locale% }

###[Update routing.yml](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/Resources/doc/index.md#step-6-import-fosuserbundle-routing-files)

Witness the power and extensibility of Symfony2's routing system

    # app/config/routing.yml
  fos_user_security:
      resource: "@FOSUserBundle/Resources/config/routing/security.xml"

  fos_user_profile:
      resource: "@FOSUserBundle/Resources/config/routing/profile.xml"
      prefix: /profile

  fos_user_register:
      resource: "@FOSUserBundle/Resources/config/routing/registration.xml"
      prefix: /register

  fos_user_resetting:
      resource: "@FOSUserBundle/Resources/config/routing/resetting.xml"
      prefix: /resetting

  fos_user_change_password:
      resource: "@FOSUserBundle/Resources/config/routing/change_password.xml"
      prefix: /profile

###Success? Not quite

We need to use our own form, Symfony2's DI will come to the rescue

    <?php

    namespace PhpMelb\BlogBundle\Form;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;
    use Symfony\Component\OptionsResolver\OptionsResolverInterface;
    use FOS\UserBundle\Form\Type\RegistrationFormType as BaseType;

    class UserType extends BaseType
    {
            /**
         * @param FormBuilderInterface $builder
         * @param array $options
         */
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            parent::buildForm($builder, $options);

          $builder
              ->add('phone')
          ;
      }

      /**
       * @param OptionsResolverInterface $resolver
       */
      public function setDefaultOptions(OptionsResolverInterface $resolver)
      {
          $resolver->setDefaults(array(
              'data_class' => 'PhpMelb\BlogBundle\Entity\User'
          ));
      }

      /**
       * @return string
       */
      public function getName()
      {
          return 'phpmelb_blogbundle_user';
      }
    }



Configure the service in the dependency injection container:

    # src/PhpMelb/BlogBundle/Resources/config/services.yml
  services:
      phpmelb.registration.form.type:
          class: PhpMelb\BlogBundle\Form\UserType
          arguments: [%fos_user.model.user.class%]
          tags:
              - { name: form.type, alias: phpmelb_blogbundle_user }
Import the services.yml file

    # app/config/config.yml
    imports:
      - { resource: @PhpMelbBlogBundle/Resources/config/services.yml }

Tell FOSUserBundle to use our FormType

    # app/config/config.yml
  fos_user:
      # ...
      registration:
          form:
              type: phpmelb_blogbundle_user

#Symfony's other strength, the ecosystem
##Capifony
```capifony .```

Adjust configuration of ```deploy.rb``` per your requirements

```cap deploy:setup```

Add the correct parameters to ```parameters.yml``` on the server

##Doctrine Extenions
```composer require stof/doctrine-extensions-bundle:~1.1```

###Update AppKernel
  // app/AppKernel.php
  public function registerBundles()
  {
      return array(
          // ...
          new Stof\DoctrineExtensionsBundle\StofDoctrineExtensionsBundle(),
          // ...
      );
  }
###Set config
  stof_doctrine_extensions:
      default_locale: en_US
      orm:
          default:
              timestampable: true

###Import traits into Entities
  use Gedmo\Timestampable\Traits\TimestampableEntity;
  use Gedmo\Mapping\Annotation as Gedmo;

  use TimestampableEntity;

```app/console doctrine:schema:update --force```

##Comments
```composer require friendsofsymfony/comment-bundle:~2.0```


    "friendsofsymfony/comment-bundle": "dev-composer as 2.1"

 and

  "repositories": [
      {
          "type": "vcs",
          "url": "https://github.com/hason/FOSCommentBundle"
      }
  ],

update ```AppKernel.php```

    <?php
  // app/AppKernel.php

  public function registerBundles()
  {
      $bundles = array(
          // ...
          new FOS\RestBundle\FOSRestBundle(),
          new FOS\CommentBundle\FOSCommentBundle(),
          new JMS\SerializerBundle\JMSSerializerBundle($this),
      );
  }
###Create ```Comment``` and ```Thread``` entities
**Comment**

    <?php
  // src/PhpMelb/BlogBundle/Entity/Comment.php

  namespace PhpMelb\BlogBundle\Entity;

  use Doctrine\ORM\Mapping as ORM;
  use FOS\CommentBundle\Entity\Comment as BaseComment;

  /**
   * @ORM\Entity
   * @ORM\ChangeTrackingPolicy("DEFERRED_EXPLICIT")
   */
  class Comment extends BaseComment
  {
      /**
       * @ORM\Id
       * @ORM\Column(type="integer")
       * @ORM\GeneratedValue(strategy="AUTO")
       */
      protected $id;

      /**
       * Thread of this comment
       *
       * @var Thread
       * @ORM\ManyToOne(targetEntity="PhpMelb\BlogBundle\Entity\Thread")
       */
      protected $thread;
  }

**Thread**

  <?php
  // src/PhpMelb/BlogBundle/Entity/Thread.php

  namespace PhpMelb\BlogBundle\Entity;

  use Doctrine\ORM\Mapping as ORM;
  use FOS\CommentBundle\Entity\Thread as BaseThread;

  /**
   * @ORM\Entity
   * @ORM\ChangeTrackingPolicy("DEFERRED_EXPLICIT")
   */
  class Thread extends BaseThread
  {
      /**
       * @var string $id
       *
       * @ORM\Id
       * @ORM\Column(type="string")
       */
      protected $id;
  }

###Update database

```app/console doctrine:schema:update --force```

###Update ```config.yml```
  # app/config/config.yml

  fos_comment:
      db_driver: orm
      class:
          model:
              comment: PhpMelb\BlogBundle\Entity\Comment
              thread: PhpMelb\BlogBundle\Entity\Thread

and remove this line from the ```assetic``` config block to enable assetic for all bundles

        bundles:        [ ]

###Add the routes to ```routing.yml```
  fos_comment_api:
      type: rest
      resource: "@FOSCommentBundle/Resources/config/routing.yml"
      prefix: /api
      defaults: { _format: html }

###Put comments on our posts!
Add this to your templates where appropriate:

    {% include 'FOSCommentBundle:Thread:async.html.twig' with {'id': entity.id } %}

###Securing comments
####Aim
* Only logged in users can comment
* Symfony2 makes this astonishingly easy

##Deploying
```cap deploy:setup```

```cap deploy```

no data

```cap database:move:to_remote```

###Fun
```composer require archer/clickatell-bundle:dev-master```

Add to AppKernel.php

    new Archer\ClickatellBundle\ArcherClickatellBundle(),

Create a message entity:

  <?php
  // src/Acme/MessageBundle/Entity/Message.php

  namespace PhpMelb\BlogBundle\Entity;

  use Archer\ClickatellBundle\Entity\Message as BaseMessage;
  use Doctrine\ORM\Mapping as ORM;

  /**
   * @ORM\Entity
   * @ORM\Table(name="clickatell_message")
   */
  class Message extends BaseMessage
  {
      /**
       * @ORM\Id
       * @ORM\Column(type="integer")
       * @ORM\GeneratedValue(strategy="AUTO")
       */
      protected $id;

      public function __construct()
      {
          parent::__construct();
          // your own logic
      }
  }

```app/console doctrine:schema:update --force```

Create an event listener:


Send me some messages